---
prev: ./structuring.md
next: ./reliability.md
---

# Масштабування II: високе навантаження

Здатність вашого бота справлятися з високим навантаженням залежить від того, як ви запускаєте бота: [через тривале опитування чи через вебхуки](../guide/deployment-types.md).
У будь-якому випадку, вам варто прочитати про деякі підводні камені [нижче](#паралелізм-складнии).

## Тривале опитування

Більшості ботів ніколи не потрібно обробляти більше кількох повідомлень на хвилину під час "пікового навантаження".
Іншими словами, масштабованість для них не має значення.
Щоб бути передбачуваним, grammY обробляє оновлення послідовно.
Ось порядок дій:

1. Отримати до 100 оновлень за допомогою `getUpdates` ([довідка Telegram Bot API](https://core.telegram.org/bots/api#getupdates)).
2. Для кожного оновлення дочекатися (`await`) виконання стеку проміжних обробників.

Проте, якщо ваш бот обробляє одне повідомлення в секунду або щось близко того під час пікових навантажень, це може почати негативно впливати на швидкість відгуку.
Наприклад, повідомлення Боба буде очікувати, поки повідомлення Аліси не буде оброблено.

Цю проблему можна вирішити, не чекаючи на завершення обробки повідомлення Аліси, тобто обробляючи обидва повідомлення одночасно.
Щоб досягти максимальної швидкості реагування, ми також хотіли б отримувати нові повідомлення, поки повідомлення Боба та Аліси ще обробляються.
В ідеалі, ми також хотіли б обмежити кількість одночасно оброблюваних повідомлень деяким фіксованим числом, щоб обмежити максимальне навантаження на сервер.

Паралельна обробка не входить до базового пакету grammY.
Замість цього, для запуску вашого бота **може бути використаний пакет [плагіну для конкурентності (runner)](../plugins/runner.md)**.
Він підтримує все вищезазначене "з коробки" і надзвичайно простий у використанні.

```ts
// Раніше
bot.start();

// За допомогою плагіну, який експортує `run`.
run(bot);
```

Початковий ліміт паралельності складає 500.
Якщо ви хочете глибше вивчити пакет, перегляньте цю [сторінку](../plugins/runner.md).

Паралелізм --- це складно, тому перегляньте [підрозділ нижче](#паралелізм-складнии), щоб дізнатися, про що слід памʼятати при використанні цього плагіну.

## Вебхуки

Якщо ви запустите бота на вебхуках, він автоматично оброблятиме оновлення паралельно, щойно вони надходитимуть.
Звичайно, для того, щоб це добре працювало під високим навантаженням, вам слід ознайомитися з [використанням вебхуків](../guide/deployment-types.md#як-використовувати-вебхуки).
Це означає, що ви все одно повинні знати про деякі наслідки паралелізму, зверніть увагу на [підрозділ нижче](#паралелізм-складнии).

Також [памʼятайте](../guide/deployment-types.md#своєчасне-завершення-запитів-вебхуків), що Telegram буде доставляти оновлення з одного чату послідовно, але оновлення з різних чатів одночасно.

## Паралелізм складний

Якщо ваш бот обробляє всі оновлення одночасно, це може спричинити низку проблем, які потребують особливої уваги.
Наприклад, якщо два повідомлення з одного чату будуть отримані одним викликом `getUpdates`, вони будуть оброблені одночасно.
Порядок повідомлень в одному чаті більше не гарантується.

Основним місцем, де це може призвести до конфлікту, є використання [сесій](../plugins/session.md), що може спричинити конфлікт запису після читання.
Уявіть собі таку послідовність подій:

1. Аліса надсилає повідомлення A.
2. Бот починає обробку A.
3. Бот зчитує дані сесії для Аліси з бази даних.
4. Аліса надсилає повідомлення B.
5. Бот починає обробку B.
6. Бот зчитує дані сесії для Аліси з бази даних.
7. Бот завершує обробку A і записує нову сесію в базу даних.
8. Бот завершує обробку B і записує нову сесію в базу даних, тим самим перезаписуючи зміни, зроблені під час обробки A.
   Втрата даних через конфлікт запису після читання!

> Примітка: ви можете спробувати використовувати транзакції бази даних для сесій, але тоді ви зможете лише виявити конфлікт, а не запобігти йому.
> Спроба використати блокування натомість ефективно усуне весь паралелізм.
> Набагато легше насамперед уникнути небезпечного запису.

Більшість інших сесійних систем серверних фреймворків просто приймають ризик станів гонок, оскільки вони трапляються не надто часто в Інтернеті.
Однак ми не хочемо цього, тому що боти Telegram набагато частіше стикаються із зіткненнями паралельних запитів на один і той самий ключ сесії.
Отже, ми повинні переконатися, що оновлення, які отримують доступ до одних і тих же даних сесії, обробляються послідовно, щоб уникнути цього небезпечного стану гонки.

Плагін для конкурентності (runner) постачається з проміжним обробником `sequentialize()`, який гарантує, що оновлення, які конфліктують, обробляються послідовно.
Ви можете налаштувати його за допомогою тієї самої функції, яку ви використовуєте для визначення ключа сесії.
Тоді він уникне вищезгаданого стану гонки, сповільнивши виключно ті оновлення, які могли б спричинити колізію.

::::code-group
:::code-group-item TypeScript

```ts
import { Bot, Context, session } from "grammy";
import { run, sequentialize } from "@grammyjs/runner";

// Створюємо бота.
const bot = new Bot("");

// Створюємо унікальний ідентифікатор для обʼєкту `Context`.
function getSessionKey(ctx: Context) {
  return ctx.chat?.id.toString();
}

// Упорядкуємо дані перед доступом до даних сесії!
bot.use(sequentialize(getSessionKey));
bot.use(session({ getSessionKey }));

// Додаємо звичайний проміжний обробник, тепер з підтримкою безпечних сесій.
bot.on("message", (ctx) => ctx.reply("Отримав ваше повідомлення."));

// Все одно запускаємо бота паралельно!
run(bot);
```

:::

:::code-group-item JavaScript

```js
const { Bot, Context, session } = require("grammy");
const { run, sequentialize } = require("@grammyjs/runner");

// Створюємо бота.
const bot = new Bot("");

// Створюємо унікальний ідентифікатор для обʼєкту `Context`.
function getSessionKey(ctx) {
  return ctx.chat?.id.toString();
}

// Упорядкуємо дані перед доступом до даних сесії!
bot.use(sequentialize(getSessionKey));
bot.use(session({ getSessionKey }));

// Додаємо звичайний проміжний обробник, тепер з підтримкою безпечних сесій.
bot.on("message", (ctx) => ctx.reply("Отримав ваше повідомлення."));

// Все одно запускаємо бота паралельно!
run(bot);
```

:::
:::code-group-item Deno

```ts
import { Bot, Context, session } from "https://deno.land/x/grammy/mod.ts";
import { run, sequentialize } from "https://deno.land/x/grammy_runner/mod.ts";

// Створюємо бота.
const bot = new Bot("");

// Створюємо унікальний ідентифікатор для обʼєкту `Context`.
function getSessionKey(ctx: Context) {
  return ctx.chat?.id.toString();
}

// Упорядкуємо дані перед доступом до даних сесії!
bot.use(sequentialize(getSessionKey));
bot.use(session({ getSessionKey }));

// Додаємо звичайний проміжний обробник, тепер з підтримкою безпечних сесій.
bot.on("message", (ctx) => ctx.reply("Отримав ваше повідомлення."));

// Все одно запускаємо бота паралельно!
run(bot);
```

:::
::::

Приєднуйтесь до [чату Telegram](https://t.me/grammyjs), щоб обговорити використання плагіну для конкурентності (runner) для вашого бота.
Ми завжди раді почути відгук від людей, які підтримують великих ботів, щоб ми могли покращити grammY на основі їхнього досвіду роботи з пакетом.
