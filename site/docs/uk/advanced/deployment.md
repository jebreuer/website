---
prev: ./proxy.md
---

# Контрольний список розгортання

Ось список речей, які слід памʼятати при розміщенні на хостингу великого бота.

> Вас також можуть зацікавити наші інструкції по розміщенню ботів.
> Перегляньте **Хостинг/Посібники** у верхній частині сторінки, щоб побачити деякі платформи, які вже мають спеціальні посібники.

## Помилки

1. [Встановіть обробник помилок за допомогою `bot.catch` для тривалого опитування або у вашому серверному фреймворку для вебхуків.](../guide/errors.md)
2. Використовуйте `await` для всіх `Promise` і встановіть **лінтинг** з правилами, які забезпечують, щоб ви ніколи про це не забували.

## Надсилання повідомлень

1. Надсилайте файли, передаючи шлях або `Buffer` замість потоку, або принаймні переконайтеся, що ви [знаєте про можливі підводні камені](./transformers.md#випадки-використання-перетворювачів).
2. Використовуйте `bot.on("callback_query:data")` як резервний обробник для [реагування на всі запити зворотного виклику](../plugins/keyboard.md#відповідь-на-натискання).
3. Використовуйтея [плагін `auto-retry`](../plugins/auto-retry.md) для автоматичного оброблення помилок, викликаних перевищенням ліміту кількості запитів, та повторного надсилання таких запитів.

## Масштабування

Це залежить від типу розгортання.

### Тривале опитування (long polling)

1. [Використовуйте плагін для конкурентності (runner).](../plugins/runner.md)
2. [Використовуйте `sequentialize` з тією самою функцією для отримання ключу сесії, що і в плагіні сесій](./scaling.md#паралелізм-складнии).
3. Перегляньте параметри конфігурації `run` ([довідка API](https://deno.land/x/grammy_runner/mod.ts?s=run)) і переконайтеся, що вони відповідають вашим потребам, або навіть розгляньте можливість зібрати власний runner зі своїм [джерелом оновлень (source)](https://deno.land/x/grammy_runner/mod.ts?s=UpdateSource) та їх [поглиначем (sink)](https://deno.land/x/grammy_runner/mod.ts?s=UpdateSink).
   Головне, що потрібно враховувати, --- це максимальне навантаження, яке ви хочете застосувати до вашого сервера, тобто скільки оновлень може бути оброблено одночасно.
4. Подумайте про реалізацію [коректного завершення роботи](./reliability.md#коректне-завершення-роботи), щоб завершити обробку поточних оновлень перш ніж ви зупините бота, наприклад, для його оновлення.

### Вебхуки

1. Переконайтеся, що ви не виконуєте довготривалі операції у проміжних обробниках: наприклад, передача великих файлів.
   [Це призводить до помилок тайм-ауту](../guide/deployment-types.md#своєчасне-завершення-запитів-вебхуків) для вебхуків та дублювання обробки оновлень, оскільки Telegram буде повторно надсилати непідтверджені оновлення.
   Замість цього розгляньте можливість використання черги завдань.
2. Ознайомтеся з конфігурацією `webhookCallback` ([довідка API](https://deno.land/x/grammy/mod.ts?s=webhookCallback)).
3. Якщо ви змінювали параметр `getSessionKey` для плагіну сесій, [використовуйте `sequentialize` з тією ж функцією для отримання ключу сесії](./scaling.md#паралелізм-складнии).
4. Якщо ви працюєте на безсерверній платформі або платформі з автоматичним масштабуванням, [встановіть інформацію про бота](https://deno.land/x/grammy/mod.ts?s=BotConfig), щоб запобігти надмірній кількості викликів `getMe`.
5. Подумайте про використання [відповідей вебхуку](../guide/deployment-types.md#відповідь-вебхуку).

## Сесії

1. Подумайте про використання `lazySessions`, як описано [тут](../plugins/session.md#ліниві-сесіі).
2. Використовуйте опцію `storage` для налаштування вашого адаптера зберігання даних, інакше всі дані будуть втрачені, коли процес бота зупиниться.

## Тестування

Пишіть тести для свого бота.
Це можна зробити за допомогою grammY наступним чином:

1. Імітуйте вихідні запити API за допомогою [перетворювачів](./transformers.md).
2. Визначте та надішліть зразки обʼєктів оновлення вашому боту за допомогою `bot.handleUpdate` ([довідка API](https://deno.land/x/grammy/mod.ts?s=Bot#method_handleUpdate_0)).
   Скористайтеся цими [обʼєктами оновлення](https://core.telegram.org/bots/webhooks#testing-your-bot-with-updates), наданими командою Telegram, для натхнення.

::: tip Запропонуйте власний фреймворк для тестування
Хоча grammY надає необхідні хуки для початку написання тестів, було б дуже корисно мати фреймворк для тестування ботів.
Це нова територія --- подібних тестових фреймворків здебільшого не існує.
Ми з нетерпінням чекаємо на ваші внески!

Приклад того, як можна провести тестування, [можна знайти тут](https://github.com/PavelPolyakov/grammy-with-tests).
:::
