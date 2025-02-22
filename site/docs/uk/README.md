---
home: true
heroImage: /images/Y.png
actions:
  - text: Розпочати
    link: /uk/guide/getting-started.html
    type: primary
  - text: Вступ
    link: /uk/guide/introduction.html
    type: secondary
features:
  - title: Простий у використанні
    details: grammY робить створення ботів Telegram настільки простим, що ви вже знаєте, як це зробити.
  - title: Гнучкий
    details: grammY відкритий і може бути розширений за допомогою плагінів, щоб точно відповідати вашим потребам.
  - title: Масштабований
    details: grammY допоможе вам, коли ваш бот стане популярним і трафік зросте.
permalink: /uk/
---

<h6 align="right">… {{ [
  'подумайте, чомУ',
  'нова ера розробки ботів',
  'швидший за вас',
  'попереду ще одне оновлення',
  'може зробити все, окрім вечері',
  'легко, як з обійстя виховати козУ',
  'обслуговано мільярди і мільярди',
][Math.floor(Math.random() * 7)] }}.</h6>

## Швидкий старт

Боти, написані на [TypeScript](https://www.typescriptlang.org) або JavaScript, працюють на різних платформах, зокрема [Node.js](https://nodejs.org).

`npm install grammy` і вставте наступний код:

::::code-group
:::code-group-item TypeScript

```ts
import { Bot } from "grammy";

const bot = new Bot(""); // <-- Помістіть токен свого бота між "" (https://t.me/BotFather)

// Відповідаємо "Привіт!" на будь-яке повідомлення.
bot.on("message", (ctx) => ctx.reply("Привіт!"));

bot.start();
```

:::
:::code-group-item JavaScript

```js
const { Bot } = require("grammy");

const bot = new Bot(""); // <-- Помістіть токен свого бота між "" (https://t.me/BotFather)

// Відповідаємо "Привіт!" на будь-яке повідомлення.
bot.on("message", (ctx) => ctx.reply("Привіт!"));

bot.start();
```

:::
:::code-group-item Deno

```ts
import { Bot } from "https://deno.land/x/grammy/mod.ts";

const bot = new Bot(""); // <-- Помістіть токен свого бота між "" (https://t.me/BotFather)

// Відповідаємо "Привіт!" на будь-яке повідомлення.
bot.on("message", (ctx) => ctx.reply("Привіт!"));

bot.start();
```

:::
::::

Працює! :tada:

---

<ClientOnly>
  <ThankYou :s="[
    'Дякуємо, ',
    '{name}',
    ', за внесок у grammY.',
    ', за створення grammY.']" />
</ClientOnly>

<div style="font-size: 0.75rem; display: flex; justify-content: center;">

© 2021-2023 &middot; grammY підтримує Telegram Bot API 6.7, який був [випущений](https://core.telegram.org/bots/api#april-21-2023) 21-го квітня 2023 року.
Остання зміна: декілька псевдонімів ботів, користувацькі емодзі та кращі inline-запити.

</div>

<ClientOnly>
  <LanguagePopup />
</ClientOnly>
