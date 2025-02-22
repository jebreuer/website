---
prev: ./flood.md
next: ./proxy.md
---

# Перетворювачі Bot API

Проміжний обробник --- це функція, яка обробляє обʼєкт контексту, тобто вхідні дані.

grammY також надає вам можливість робити протилежне.
_Перетворювач_ --- це функція, яка обробляє вихідні дані, тобто:

- назву методу API бота, який потрібно викликати,
- обʼєкт вмісту запиту, що відповідає методу.

Замість того, щоб використовувати `next` як останній аргумент для виклику наступного проміжного обробника, ви отримуєте `prev` як перший аргумент для виклику наступних перетворювачів.
Перегляньте сигнатуру типу `Transformer` ([довідка API grammY](https://deno.land/x/grammy/mod.ts?s=Transformer)), і ви побачите, як вона відповідає цьому.
Зауважте, що `Payload<M, R>` посилається на обʼєкт вмісту запиту, який має відповідати даному методу, а `ApiResponse<ApiCallResult<M, R>>` це тип результату, що повертається викликаним методом.

Останній викликаний перетворювач є вбудованим викликом, який виконує такі дії, як серіалізація JSON певних полів і, врешті-решт, виклик `fetch`.

Для перетворювачів не існує еквівалента класу `Composer`, оскільки це, мабуть, надмірність, але, якщо він вам потрібен, ви можете написати власний.
PR вітається! :wink:

## Встановлення перетворювача

Перетворювач можна встановити на `bot.api`.
Ось приклад для перетворювача, який нічого не робить:

```ts
// Перетворювач, який нічого не робить
bot.api.config.use((prev, method, payload, signal) =>
  prev(method, payload, signal)
);

// Порівняємо з таким самим марним проміжним обробником
bot.use((ctx, next) => next());
```

Ось приклад перетворювача, який запобігає всім викликам API:

```ts
// Неправильно повертаємо `undefined` замість відповідних типів обʼєктів.
bot.api.config.use((prev, method, payload) => undefined as any);
```

Ви також можете встановити перетворювачі на обʼєкті API обʼєкта контексту.
Після цього перетворювач буде використовуватися лише тимчасово для запитів API, які виконуються для цього конкретного обʼєкта контексту.
Виклики `bot.api` залишаються без змін.
Виклики через обʼєкти контексту паралельно запущеного проміжного обробника також не зачіпаються.
Як тільки відповідний проміжний обробник завершує роботу, перетворювач буде відкинуто.

```ts
bot.on("message", async (ctx) => {
  // Встановимо на всі обʼєкти контексту, які обробляють повідомлення.
  ctx.api.config.use((prev, method, payload, signal) =>
    prev(method, payload, signal)
  );
});
```

> Параметр `signal` завжди слід передавати в `prev`.
> Він дозволяє скасовувати запити і є важливим для роботи `bot.stop`.

Перетворювачі, встановлені на `bot.api`, будуть попередньо встановлені на кожному обʼєкті `ctx.api`.
Отже, виклики до `ctx.api` будуть перетворені як перетворювачами, встановленими на `ctx.api`, так і перетворювачами, встановленими на `bot.api`.

## Випадки використання перетворювачів

Перетворювачі настільки ж гнучкі, як і проміжні обробники, і вони мають так само багато різних застосувань.

Наприклад, [плагін для інтерактивних меню](../plugins/menu.md) встановлює перетворювач для перетворення вихідних екземплярів меню на відповідний вміст запиту.
Ви також можете використовувати їх для

- реалізації [обмеження запитів](../plugins/transformer-throttler.md),
- імітування запитів до API під час тестування,
- додавання [повторення запитів](../plugins/auto-retry.md),
- багато чого іншого.

Зауважте, однак, що повторний виклик API може мати дивні побічні ефекти: якщо ви викликаєте `sendDocument` і передаєте екземпляр потоку читання в `InputFile`, то потік буде прочитано при першій спробі запиту.
Якщо ви знову викличете `prev`, потік може вже бути частково використано, що призведе до пошкодження файлів.
Тому надійніше передавати шлях до файлу до `InputFile`, щоб grammY за потреби міг відтворити потік.

## Розширювач для API

grammY має [розширювач для контексту](../guide/context.md#розширювач-для-контексту), за допомогою яких можна налаштувати тип контексту.
Сюди входять методи API: як ті, що знаходяться безпосередньо на обʼєкті контексту: `ctx.reply` тощо, так і всі методи в `ctx.api` і `ctx.api.raw`.
Однак ви не можете змінювати типи `bot.api` та `bot.api.raw` за допомогою розширювача для контексту.

Ось чому grammY підтримує _розширювачі для API_.
Вони вирішують цю проблему:

```ts
import { Api, Bot, Context } from "grammy";
import { SomeApiFlavor, SomeContextFlavor, somePlugin } from "some-plugin";

// Розширювач для контексту
type MyContext = Context & SomeContextFlavor;
// Розширювач для API
type MyApi = Api & SomeApiFlavor;

// Використовуємо обидва типи розширювачів
const bot = new Bot<MyContext, MyApi>("");

// Встановлюємо певний плагін
bot.api.config.use(somePlugin());

// Тепер викличемо `bot.api` зі зміненими типами з розширювачем для API.
bot.api.somePluginMethod();

// Також використаємо налаштований тип контексту з розширювачем для контексту.
bot.on("message", (ctx) => ctx.api.somePluginMethod());
```

Розширювачи для API працюють точно так само, як і розширювачи для контексту.
Існують як додавальні, так і перетворювальні розширювачи для API, а кілька розширювачів для API можна комбінувати так само, як і розширювачі для контексту.
Якщо ви не впевнені, як це працює, поверніться до [розділу про розширювачи для контексту](../guide/context.md#розширювач-для-контексту) у посібнику.
