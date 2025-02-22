# Емодзі (`emoji`)

За допомогою цього плагіна ви можете легко вставляти емодзі у свої відповіді, шукаючи їх безпосередньо у редакторі коду, замість того, щоб вручну копіювати і вставляти емодзі з Інтернету у свій код.

## Чому я маю його використовувати?

Чому б ні? Люди постійно використовують емодзі у своєму коді, щоб проілюструвати повідомлення, яке вони хочуть надіслати, або щоб упорядкувати дещо.
Але ви щоразу втрачаєте фокус, коли вам потрібен новий емодзі:

1. Ви припиняєте написання коду, щоб розпочати пошук конкретного емодзі.
2. Ви заходите в чат Telegram і витрачаєте близько 6-ти секунд, якщо не більше, на пошук потрібного вам емодзі.
3. Ви копіюєте його у свій код і повертаєтесь до написання коду, але з втраченим фокусом.

З цим плагіном ви не тільки не припиняєте писати код, але й не втрачаєте фокус.
Існують також системи та/або редактори, які не люблять й не показують емодзі, тому в результаті ви вставляєте білий квадрат на кшталт цього: `Я такий щасливий □`.

Цей плагін має на меті вирішити ці проблеми, взявши на себе складне завдання парсингу емодзі у різноманітних системах і дозволяючи вам просто шукати їх у зручний спосіб за допомогою автодоповнення. Тепер вищеописані кроки можна звести до цього:

1. Опишіть емодзі, який ви хочете, та використовуйте його. Прямо у вашому коді. Ось так просто.

### Це що, чаклунство?

Ні, це називається шаблонними рядками.
Ви можете прочитати про них докладніше [тут](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

## Встановлення та приклади

Ви можете встановити цей плагін на свого бота наступним чином:

::::code-group
:::code-group-item TypeScript

```ts
import { Bot, Context } from "grammy";
import { EmojiFlavor, emojiParser } from "@grammyjs/emoji";

// Це називається розширювач для контексту
// Ви можете прочитати про це докладніше за посиланням:
// https://grammy.dev/uk/guide/context.html#перетворювальнии-розширювач
type MyContext = EmojiFlavor<Context>;

const bot = new Bot<MyContext>("");

bot.use(emojiParser());
```

:::
:::code-group-item JavaScript

```js
const { Bot } = require("grammy");
const { emojiParser } = require("@grammyjs/emoji");

const bot = new Bot("");

bot.use(emojiParser());
```

:::
:::code-group-item Deno

```ts
import { Bot, Context } from "https://deno.land/x/grammy/mod.ts";
import {
  EmojiFlavor,
  emojiParser,
} from "https://deno.land/x/grammy_emoji/mod.ts";

// Це називається розширювач для контексту
// Ви можете прочитати про це докладніше за посиланням:
// https://grammy.dev/uk/guide/context.html#перетворювальнии-розширювач
type MyContext = EmojiFlavor<Context>;

const bot = new Bot<MyContext>("");

bot.use(emojiParser());
```

:::
::::

Тепер ви можете отримувати емодзі за їхніми назвами:

```js
bot.command("start", async (ctx) => {
  const parsedString = ctx.emoji`Вітаємо! ${"smiling_face_with_sunglasses"}`; // => Вітаємо! 😎
  await ctx.reply(parsedString);
});
```

Крім того, ви можете відповісти безпосередньо за допомогою методу `replyWithEmoji`:

```js
bot.command("ping", async (ctx) => {
  await ctx.replyWithEmoji`Понг ${"ping_pong"}`; // => Понг 🏓
});
```

::: warning Майте на увазі, що
Методи `ctx.emoji` та `ctx.replyWithEmoji` **ЗАВЖДИ** використовують шаблонні рядки.
Якщо ви не знайомі з цим синтаксисом, ви можете прочитати про нього докладніше [тут](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).
:::

## Загальні відомості про плагін

- Назва: `emoji`
- Джерело: <https://github.com/grammyjs/emoji>
- Довідка: <https://deno.land/x/grammy_emoji/mod.ts>
