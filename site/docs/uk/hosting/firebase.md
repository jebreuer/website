# Хостинг: Firebase Functions

Цей посібник допоможе вам розгорнути вашого бота на [Firebase Functions](https://firebase.google.com/docs/functions).

## Передумови

Щоб скористатися цим посібником, вам потрібен обліковий запис Google.
Якщо у вас його ще немає, ви можете створити його [тут](https://accounts.google.com/signup).

## Налаштування

Цей розділ проведе вас через процес налаштування.
Якщо вам потрібні більш детальні пояснення кожного кроку, зверніться до [офіційної документації Firebase](https://firebase.google.com/docs/functions/get-started).

### Створення проєкту Firebase

1. Перейдіть до [панелі управління Firebase](https://console.firebase.google.com/) та натисніть **Add project**.
2. Можливо, вам буде запропоновано переглянути умови використання Firebase. Перегляньте та прийміть їх.
3. Натисніть **Continue**.
4. Вирішіть, чи бажаєте ви надавати дані аналітики чи ні.
5. Натисніть **Create Project**.

### Налаштування проєкту

Щоб писати функції та розгортати їх в середовищі Firebase Functions, вам потрібно налаштувати середовище Node.js та встановити Firebase CLI.

> Важливо зауважити, що на даний момент Firebase Functions підтримує версії Node.js 14, 16 та 18.
> Для отримання додаткової інформації про підтримувані версії Node.js зверніться [сюди](https://firebase.google.com/docs/functions/manage-functions?hl=ru#set_nodejs_version).

Після встановлення Node.js та NPM, глобально встановіть Firebase CLI:

```sh
npm install -g firebase-tools
```

### Ініціалізація проєкту

1. Виконайте команду `firebase login`, щоб відкрити браузер та аутентифікуватися з допомогою Firebase CLI зі своїм обліковим записом.
2. За допомогою команди `cd` перейдіть до каталогу свого проєкту.
3. Виконайте команду `firebase init functions` та введіть `y`, коли вас запитають, чи хочете ви ініціалізувати нову кодову базу.
4. Виберіть `use existing project` та оберіть проєкт, який ви створили на 1-у кроці.
5. CLI дає вам два варіанти підтримки мов:
   - JavaScript
   - TypeScript
6. За потреби ви можете вибрати ESLint.
7. CLI запитає, чи хочете ви встановити залежності за допомогою npm.
   Якщо ви використовуєте інший менеджер пакетів, наприклад, `yarn` або `pnpm`, ви можете відхилити запит.
   У такому випадку вам потрібно буде перейти до каталогу `functions` за допомогою команди `cd` та встановити залежності вручну.
8. Відкрийте `./functions/package.json` та знайдіть ключ: `"engines": {"node": "16"}`.
   Версія `node` повинна відповідати версії Node.js, яку ви встановили.
   Інакше проєкт може не запуститися.

## Підготовка коду

Ви можете використати цей короткий приклад як точку старту:

```ts
import * as functions from "firebase-functions";
import { Bot, webhookCallback } from "grammy";

const bot = new Bot("");

bot.command("start", (ctx) => ctx.reply("Ласкаво просимо! Все працює."));
bot.command("ping", (ctx) => ctx.reply(`Понг! ${new Date()}`));

// Під час розробки ви можете запускати свою функцію за допомогою https://localhost/<firebase-проєкт>/us-central1/helloWorld
export const helloWorld = functions.https.onRequest(webhookCallback(bot));
```

## Локальна розробка

Під час розробки ви можете використовувати набір емуляторів Firebase, щоб запускати свій код локально.
Це значно швидше, ніж розгортати кожну зміну на Firebase.
Щоб встановити емулятори, виконайте наступну команду:

```sh
firebase init emulators
```

Емулятор функцій повинен бути вже вибраний.
Якщо він не вибраний, перейдіть до нього за допомогою клавіш зі стрілками та виберіть його, використовуючи `пробіл`.
Щодо запитань про те, який порт використовувати для кожного емулятора, просто натисніть `Enter`.

Щоб запустити емулятори та виконати свій код, скористайтеся командою:

```sh
npm run serve
```

::: tip
З якоїсь причини стандартна конфігурація сценарію npm не запускає компілятор TypeScript в режимі спостереження.
Отже, якщо ви використовуєте TypeScript, вам також потрібно запустити:

```sh
npm run build:watch
```

:::

Після запуску емуляторів ви повинні знайти рядок у виводі консолі, який виглядає наступним чином:

```sh
+  functions[us-central1-helloWorld]: http function initialized (http://127.0.0.1:5001/<firebase-проєкт>/us-central1/helloWorld).
```

Це локальна URL-адреса вашої хмарної функції.
Однак ваша функція доступна лише локально на вашому компʼютері.
Щоб перевірити свого бота, вам потрібно викласти вашу функцію в Інтернеті, аби Telegram API міг відправляти оновлення вашому боту.
Існує кілька сервісів, таких як [localtunnel](https://localtunnel.me) або [ngrok](https://ngrok.com), які можуть вам у цьому допомогти.
У цьому прикладі ми будемо використовувати localtunnel.

Спочатку встановіть localtunnel:

```sh
npm i -g localtunnel
```

Після цього ви можете перенаправити порт `5001`:

```sh
lt --port 5001
```

localtunnel повинен надати вам унікальну URL-адресу, наприклад, `https://modern-heads-sink-80-132-166-120.loca.lt`.

Все, що залишилося --- це повідомити Telegram, куди надсилати оновлення.
Ви можете зробити це, викликавши `setWebhook`.
Наприклад, відкрийте нову вкладку у своєму браузері та відвідайте цю адресу:

```text
https://api.telegram.org/bot<токен-бота>/setWebhook?url=<адреса>/<firebase-проєкт>/us-central1/helloWorld
```

Замініть `<токен-бота>` на свій дійсний токен бота й `<адреса>` на власну URL-адресу, яку ви отримали від localtunnel.

Ви повинні побачити це у вікні свого браузера.

```json
{
  "ok": true,
  "result": true,
  "description": "Webhook was set"
}
```

Наразі ваш бот готовий для тестування розгортання.

## Розгортання

Щоб розгорнути вашу функцію, просто запустіть:

```sh
firebase deploy
```

Firebase CLI надасть вам URL-адресу вашої функції одразу після того, як розгортання буде завершено.
Посилання повинно виглядати якось так: `https://<регіон>.<назва-проєкту.cloudfunctions.net/helloWorld`.
Для отримання докладнішої інформації ви можете звернутися до 8-го кроку в [посібнику початку роботи](https://firebase.google.com/docs/functions/get-started?hl=ru#deploy-functions-to-a-production-environment).

Після розгортання вам потрібно повідомити Telegram, куди надсилати оновлення для вашого бота, викликавши метод `setWebhook`.
Для цього відкрийте нову вкладку в браузері і відвідайте цю адресу:

```text
https://api.telegram.org/bot<токен-бота>/setWebhook?url=https://<регіон>.<назва-проєкту>.cloudfunctions.net/helloWorld
```

Замініть `<токен-бота>` на токен вашого бота, `<регіон>` на назву регіону, де ви розгорнули свою функцію, і `<назва-проєкту>` на назву вашого проєкту Firebase.
Firebase CLI повинен надати вам повну URL-адресу вашої хмарної функції, тому ви можете просто вставити її після параметра `?url=` в методі `setWebhook`.

Якщо все налаштовано правильно, ви повинні побачити цю відповідь у вікні вашого браузера:

```json
{
  "ok": true,
  "result": true,
  "description": "Webhook was set"
}
```

Це все, бот готовий до роботи.
Перейдіть до Telegram та подивіться, як бот відповідає на повідомлення!
