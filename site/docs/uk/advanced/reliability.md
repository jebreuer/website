---
prev: ./scaling.md
next: ./flood.md
---

# Масштабування III: надійність

Якщо ви переконалися, що для вашого бота створено належну [обробку помилок](../guide/errors.md), то все готово до запуску.
Усі помилки, яких слід очікувати: невдалі виклики API, невдалі мережеві запити, невдалі запити до бази даних, невдалі проміжні обробники тощо, відловлюються.

Ви повинні переконатися, що завжди очікуєте (`await`) всі `Promise` або, принаймні, викликаєте `catch` на них для обробки помилок, якщо ви не хочете використовувати `await`.
Використовуйте відповідне правило для лінтингу, щоб переконатися, що ви не забудете про це.

## Коректне завершення роботи

Для ботів, які використовують тривале опитування, є ще один момент, який слід врахувати.
Коли ви збираєтеся зупинити ваш екземпляр під час роботи, вам слід перехоплювати події `SIGTERM` і `SIGINT` й викликати вбудований метод `bot.stop` або зупинити бота за допомогою його [обробника](https://deno.land/x/grammy_runner/mod.ts?s=RunnerHandle#prop_stop), якщо використовується плагін для конкурентності (runner):

### Просте тривале опитування

::::code-group

:::code-group-item TypeScript

```ts
import { Bot } from "grammy";

const bot = new Bot("");

// Зупиняємо бота, коли процес Node.js
// наближається до завершення
process.once("SIGINT", () => bot.stop());
process.once("SIGTERM", () => bot.stop());

await bot.start();
```

:::

:::code-group-item JavaScript

```js
const { Bot } = require("grammy");

const bot = new Bot("");

// Зупиняємо бота, коли процес Node.js
// наближається до завершення
process.once("SIGINT", () => bot.stop());
process.once("SIGTERM", () => bot.stop());

await bot.start();
```

:::

:::code-group-item Deno

```ts
import { Bot } from "https://deno.land/x/grammy/mod.ts";

const bot = new Bot("");

// Зупиняємо бота, коли процес Deno
// наближається до завершення
Deno.addSignalListener("SIGINT", () => bot.stop());
Deno.addSignalListener("SIGTERM", () => bot.stop());

await bot.start();
```

:::
::::

### Використовуючи плагін для конкурентості (runner)

::::code-group

:::code-group-item TypeScript

```ts
import { Bot } from "grammy";
import { run } from "@grammyjs/runner";

const bot = new Bot("");

const runner = run(bot);

// Зупиняємо бота, коли процес Node.js
// наближається до завершення
const stopRunner = () => runner.isRunning() && runner.stop();
process.once("SIGINT", stopRunner);
process.once("SIGTERM", stopRunner);
```

:::

:::code-group-item JavaScript

```js
const { Bot } = require("grammy");
const { run } = require("@grammyjs/runner");

const bot = new Bot("");

const runner = run(bot);

// Зупиняємо бота, коли процес Node.js
// наближається до завершення
const stopRunner = () => runner.isRunning() && runner.stop();
process.once("SIGINT", stopRunner);
process.once("SIGTERM", stopRunner);
```

:::
:::code-group-item Deno

```ts
import { Bot } from "https://deno.land/x/grammy/mod.ts";
import { run } from "https://deno.land/x/grammy_runner/mod.ts";

const bot = new Bot("");

const runner = run(bot);

// Зупиняємо бота, коли процес Deno
// наближається до завершення
const stopRunner = () => runner.isRunning() && runner.stop();
Deno.addSignalListener("SIGINT", stopRunner);
Deno.addSignalListener("SIGTERM", stopRunner);
```

:::
::::

По великому рахунку, це все, що стосується надійності; зараз ваш екземпляр повинен:registered: ніколи:tm: не падати.

## Гарантії надійності

Що робити, якщо ваш бот обробляє фінансові транзакції, і ви повинні розглянути сценарій [`kill -9`](https://stackoverflow.com/questions/43724467/what-is-the-difference-between-kill-and-kill-9), коли процесор фізично виходить з ладу або відбувається відключення електроенергії в дата-центрі?
Якщо з якихось причин хтось або щось фактично жорстко вбиває процес, це стає трохи складнішим.

По суті, боти не можуть гарантувати виконання вашого проміжного обробника _рівно один раз_.
Прочитайте цю [дискусію](https://github.com/tdlib/telegram-bot-api/issues/126) на GitHub, щоб дізнатися більше про те, **чому** ваш бот може надсилати дублікати повідомлень або взагалі не надсилати у вкрай рідкісних випадках.
Решта цього розділу присвячена тому, як поводиться grammY за таких незвичних обставин і як впоратися з цими ситуаціями.

> Вам просто цікаво написати бота Telegram? [Пропустіть решту цієї сторінки.](./flood.md)

### Вебхук

Якщо ви запускаєте бота на вебхуках, сервер Bot API повторить спробу доставити оновлення вашому боту, якщо він не відповість `OK` вчасно.
Це значною мірою визначає поведінку системи --- якщо вам потрібно запобігти обробці дублікатів оновлень, вам слід створити власну дедуплікацію на основі `update_id`.
grammY не робить цього для вас, але не соромтеся робити PR, якщо вважаєте, що це буде корисно ще для когось.

### Тривале опитування

Тривале опитування цікавіше.
Вбудоване опитування в основному повторно запускає останній пакет оновлень, який було отримано, але не вдалося завершити.

> Зверніть увагу, що якщо ви правильно зупините бота за допомогою `bot.stop`, [зсув оновлень](https://core.telegram.org/bots/api#getting-updates) буде синхронізовано з серверами Telegram шляхом виклику `getUpdates` з правильним зміщенням, але без обробки даних оновлення.

Іншими словами, ви ніколи не втратите жодного оновлення, проте може статися так, що ви повторно опрацюєте до 100 оновлень, які бачили раніше.
Оскільки виклик `sendMessage` не є ідемпотентним, користувачі можуть отримувати дублікати повідомлень від вашого бота.
Однак, _принаймні один раз_ обробка гарантується.

### Плагін для конкурентності (runner)

Якщо ви використовуєте [плагін для конкурентності (runner)](../plugins/runner.md) у паралельному режимі, наступний виклик `getUpdates` може бути виконано до того, як ваші проміжні обробники оброблять перше оновлення поточної пачки.
Таким чином, зміщення оновлення буде [підтверджено](https://core.telegram.org/bots/api#getupdates) передчасно.
Це вартість потужного паралелізму, і, на жаль, її неможливо уникнути без зниження пропускної здатності та швидкості реагування.
Унаслідок цього, якщо ваш екземпляр буде вбито в потрібний чи не потрібний момент, може статися так, що до 100 оновлень не вдасться отримати знову, оскільки Telegram вважатиме їх вже підтвердженими.
Це призведе до втрати даних.

Якщо дуже важливо запобігти цьому, вам слід скористатися джерелами оновлень (source) та їх поглиначами (sink) з пакету плагіна для конкурентності (runner) для створення власного способу обробки оновлень, який спершу пропускатиме всі оновлення через чергу повідомлень.

1. Вам потрібно створити [поглинач (sink)](https://deno.land/x/grammy_runner/mod.ts?s=UpdateSink), який буде поміщати оновлення у чергу, і запустити один runner, який буде обслуговувати цю чергу повідомлень.
2. Після цього вам треба створити [джерело оновлень (source)](https://deno.land/x/grammy_runner/mod.ts?s=UpdateSource), яке буде отримувати оновлення з цієї черги повідомлень.
   Отже, ви фактично запустите два окремих екземпляри плагіну для конкурентності (runner).

Наскільки нам відомо, цей розпливчастий проєкт, описаний вище, був лише задуманий, але не реалізований.
Будь ласка, [звʼяжіться з групою в Telegram](https://t.me/grammyjs), якщо у вас виникнуть запитання або якщо ви спробуєте це зробити і зможете поділитися своїми успіхами.

З іншого боку, якщо ваш бот знаходиться під великим навантаженням і опитування оновлень сповільнюється через [автоматичні обмеження навантаження](../plugins/runner.md#поглинач), зростає ймовірність того, що деякі оновлення будуть отримані повторно, що знову призведе до появи дублікатів повідомлень.
Отже, ціна повного паралелізму полягає в тому, що не можна гарантувати обробку ні _щонайменше один раз_, ні _щонайбільше один раз_.
