---
home: true
heroImage: /images/Y.png
actions:
  - text: Get Started
    link: /guide/getting-started.html
    type: primary
  - text: Introduction
    link: /guide/introduction.html
    type: secondary
features:
  - title: Easy to Use
    details: grammY makes creating Telegram bots so simple you already know how to do it.
  - title: Flexible
    details: grammY is open and can be extended by plugins to make it fit exactly your needs.
  - title: Scalable
    details: grammY has you covered when your bot gets popular and the traffic increases.
permalink: /
---

<h6 align="right">… {{ [
  'think of the whY',
  'a new era of bot development',
  'runs faster than you',
  'one update ahead',
  'can do anything except dishes',
  'easy peasy lemon squeezY',
  'billions and billions served',
][Math.floor(Math.random() * 7)] }}.</h6>

## Quickstart

Bots are written in [TypeScript](https://www.typescriptlang.org) (or JavaScript) and run on various platforms, including [Node.js](https://nodejs.org).

`npm install grammy` and paste the following code:

::::code-group
:::code-group-item TypeScript

```ts
import { Bot } from "grammy";

const bot = new Bot(""); // <-- put your bot token between the "" (https://t.me/BotFather)

// Reply to any message with "Hi there!".
bot.on("message", (ctx) => ctx.reply("Hi there!"));

bot.start();
```

:::
:::code-group-item JavaScript

```js
const { Bot } = require("grammy");

const bot = new Bot(""); // <-- put your bot token between the "" (https://t.me/BotFather)

// Reply to any message with "Hi there!".
bot.on("message", (ctx) => ctx.reply("Hi there!"));

bot.start();
```

:::
:::code-group-item Deno

```ts
import { Bot } from "https://deno.land/x/grammy/mod.ts";

const bot = new Bot(""); // <-- put your bot token between the "" (https://t.me/BotFather)

// Reply to any message with "Hi there!".
bot.on("message", (ctx) => ctx.reply("Hi there!"));

bot.start();
```

:::
::::

Works! :tada:

---

<ClientOnly>
  <ThankYou :s="[
    'Thank you, ',
    '{name}',
    ', for being a contributor to grammY.',
    ', for creating grammY.'
  ]" />
</ClientOnly>

<div style="font-size: 0.75rem; display: flex; justify-content: center;">

© 2021-2023 &middot; grammY supports Telegram Bot API 6.7 which was [released](https://core.telegram.org/bots/api#april-21-2023) on April 21, 2023.
(Last highlight: multiple bot names, custom emoji, and better inline queries)

</div>

<ClientOnly>
  <LanguagePopup />
</ClientOnly>
