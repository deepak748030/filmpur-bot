Absolutely — I *will* provide the latest **Telegraf (npm) documentation and how-to-use guide**, **section by section**, and you’ll get the *next part* when you write **“next”**. I will *not* dump irrelevant Reddit threads — only **official, up-to-date Telegraf docs and examples**.

Before we proceed: most people *wrongly* think Telegraf is abandoned. That’s incorrect — **v4.x is the latest major version** and it supports the current Telegram Bot API plus TypeScript typings. ([Telegraf][1])

---

# 📘 **TELEGRAPH (npm) — Official Documentation & Usage Guide**

*(Latest stable release: v4.x)* ([Telegraf][1])

---

## ✅ **1. What is Telegraf**

Telegraf is a **modern Telegram Bot framework for Node.js**, designed to simplify building Telegram bots in JavaScript or TypeScript.
It supports the full Telegram Bot API, has strong typing, middleware support, extensibility, and works with webhooks or long polling. ([Telegraf][1])

---

## ✅ **2. Telegram Bot Token**

Before writing code, you must get your bot token:

✔ Open Telegram
✔ Chat with **BotFather**
✔ Use `/newbot` to create a bot
✔ BotFather provides a **TOKEN** like:

```
123456789:AbCdefGhIJKlmNoPQRsTUVwxyZ
```

You will use this token in your code. ([Telegraf][1])

---

## ✅ **3. Installation**

Install Telegraf into your Node.js project:

```bash
npm install telegraf
# or
yarn add telegraf
# or
pnpm add telegraf
```

This adds the framework as a dependency. ([Telegraf][1])

---

## ✅ **4. Basic Bot Example (Core Usage)**

This is *the minimal working bot*:

```js
const { Telegraf } = require('telegraf');

const bot = new Telegraf(process.env.BOT_TOKEN);

bot.start((ctx) => ctx.reply('Welcome'));
bot.help((ctx) => ctx.reply('Send me a sticker'));
bot.on('sticker', (ctx) => ctx.reply('👍'));
bot.hears('hi', (ctx) => ctx.reply('Hey there'));

bot.launch();

process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));
```

Explanation:

* `bot.start()` handles `/start`
* `bot.help()` handles `/help`
* `bot.on('sticker')` handles sticker messages
* `bot.hears('hi')` listens for text **hi**
* `bot.launch()` starts the bot
* Graceful stop handlers prevent crashes on shutdown

This is canonical usage. ([npm.io][2])

---

## ✅ **5. Understanding `ctx` (Context)**

Every update comes with a `ctx` object — short for *context*.

`ctx` contains:

* `ctx.update` — raw Telegram update
* `ctx.telegram` — low-level API calls
* `ctx.reply()` — shorthand reply method
* And other helpers

Example:

```js
bot.on('text', (ctx) => {
  ctx.reply(`You said: ${ctx.message.text}`);
});
```

This simplifies message handling. ([npm.io][2])

---

## ✅ **6. Middleware System**

Telegraf’s middleware model resembles Koa:

```js
bot.use(async (ctx, next) => {
  console.log('Before');
  await next();   // next middleware
  console.log('After');
});
```

Middleware lets you:

✔ Log updates
✔ Modify context
✔ Conditionally handle actions
✔ Build reusable components

When calling `await next()`, the next middleware runs. ([npm.io][2])

---

## ✅ **7. Commands, Filters and Actions**

### 🟡 Commands

```js
bot.command('start', (ctx) => {
  ctx.reply('Welcome!');
});
```

### 🟡 Filters (by type)

```js
const { message } = require('telegraf/filters')
bot.on(message('text'), ctx => ctx.reply('Text received!'));
```

### 🟡 Actions (Callback Queries)

```js
bot.action('btn_click', (ctx) => ctx.reply('Button clicked!'));
```

These cover typical bot input types. ([npm.io][2])

---

## ✅ **8. TypeScript Support**

Telegraf is written in TypeScript and supports strong typing.

You can *extend context*:

```ts
import { Telegraf, Context } from 'telegraf';

interface MyContext extends Context {
  userRole?: string;
}

const bot = new Telegraf<MyContext>(TOKEN);

bot.use((ctx, next) => {
  ctx.userRole = 'admin';
  return next();
});
```

This enables proper typing for your custom data. ([npm.io][2])

---

## ✅ **9. Launching the Bot**

Two modes:

#### 🔹 Polling (default)

```js
bot.launch();
```

#### 🔹 Webhooks (production)

You configure a webhook with a URL:

```js
bot.launch({
  webhook: {
    domain: 'https://yourdomain.com',
    port: process.env.PORT,
  }
});
```

Use webhooks for scalable production bots.

---

## ✅ **10. Graceful Shutdown**

To avoid lost updates:

```js
process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));
```

This is required in production environments. ([npm.io][2])

---

## 📌 **You Asked For — Latest Official Documentation Only**

This content is taken from:

🔹 npm registry info for Telegraf (latest)
🔹 Official telegraf.js docs site
🔹 GitHub repo information
(All reflect the current stable v4.* series) ([Telegraf][1])

---

If you want the **next section** (deeper API reference: context methods, telegram API calls, keyboards, inline buttons, sessions, scenes, error handling, deployment configuration), just type:

👉 **next**

[1]: https://telegraf.js.org/index.html?utm_source=chatgpt.com "telegraf.js - v4.16.3"
[2]: https://npm.io/package/telegraf?utm_source=chatgpt.com "Telegraf NPM | npm.io"
