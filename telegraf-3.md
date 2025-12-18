Good. Now we move from **toy bots** to **real production bots**.
If you skip this part in real projects, your bot will become unmaintainable very fast.

Everything below is **latest Telegraf v4.x only**.

---

# 📘 **TELEGRAF (npm) – PART 3**

## Sessions, Scenes, Wizards, State Management

---

## 25️⃣ Why you NEED sessions (hard truth)

If your bot asks:

* “Send phone number”
* then “Send OTP”
* then “Confirm”

…and you don’t use sessions — your logic **will break**.

> Telegram is stateless.
> **You must manage state yourself.**

That’s what **sessions** are for.

---

## 26️⃣ Built-in Session Middleware

### Install (already included in Telegraf v4)

No extra package needed.

```js
const { Telegraf, session } = require('telegraf');

const bot = new Telegraf(process.env.BOT_TOKEN);

bot.use(session());
```

This adds:

```js
ctx.session
```

⚠ Session is **per user per chat**.

---

## 27️⃣ Basic Session Example

```js
bot.command('start', (ctx) => {
  ctx.session.step = 1;
  ctx.reply('What is your name?');
});

bot.on('text', (ctx) => {
  if (ctx.session.step === 1) {
    ctx.session.name = ctx.message.text;
    ctx.session.step = 2;
    ctx.reply('How old are you?');
  } else if (ctx.session.step === 2) {
    ctx.session.age = ctx.message.text;
    ctx.reply(
      `Name: ${ctx.session.name}\nAge: ${ctx.session.age}`
    );
    ctx.session = null; // clear
  }
});
```

### Reality check

This works, but:

* Gets messy
* Hard to scale
* Hard to debug

**That’s why Scenes exist.**

---

## 28️⃣ Scenes & Stage (Structured Conversations)

Scenes allow **clean multi-step flows**.

### Import

```js
const { Scenes, session } = require('telegraf');
```

---

## 29️⃣ Creating a Scene

```js
const stepScene = new Scenes.BaseScene('stepScene');

stepScene.enter((ctx) => ctx.reply('Enter your name'));
stepScene.on('text', (ctx) => {
  ctx.scene.state.name = ctx.message.text;
  ctx.reply('Enter your age');
  ctx.scene.enter('ageScene');
});
```

---

## 30️⃣ Another Scene

```js
const ageScene = new Scenes.BaseScene('ageScene');

ageScene.on('text', (ctx) => {
  const age = ctx.message.text;
  ctx.reply(
    `Name: ${ctx.scene.state.name}, Age: ${age}`
  );
  ctx.scene.leave();
});
```

---

## 31️⃣ Registering Scenes

```js
const stage = new Scenes.Stage([stepScene, ageScene]);

bot.use(session());
bot.use(stage.middleware());

bot.command('start', (ctx) => {
  ctx.scene.enter('stepScene');
});
```

✔ Clean
✔ Predictable
✔ Scalable

---

## 32️⃣ Wizard Scenes (Best for Linear Flows)

Wizard is a **step-by-step flow manager**.

### Example: Registration Wizard

```js
const { Scenes } = require('telegraf');

const wizard = new Scenes.WizardScene(
  'register',
  (ctx) => {
    ctx.reply('Enter name');
    return ctx.wizard.next();
  },
  (ctx) => {
    ctx.wizard.state.name = ctx.message.text;
    ctx.reply('Enter email');
    return ctx.wizard.next();
  },
  (ctx) => {
    ctx.reply(
      `Saved:\n${ctx.wizard.state.name}\n${ctx.message.text}`
    );
    return ctx.scene.leave();
  }
);
```

---

## 33️⃣ Wizard Setup

```js
const stage = new Scenes.Stage([wizard]);

bot.use(session());
bot.use(stage.middleware());

bot.command('register', (ctx) => ctx.scene.enter('register'));
```

**Use Wizard when:**

* Order matters
* Linear forms
* Onboarding flows

---

## 34️⃣ Scene State vs Session State

| Feature            | Scope              |
| ------------------ | ------------------ |
| `ctx.session`      | Global per chat    |
| `ctx.scene.state`  | Only current scene |
| `ctx.wizard.state` | Wizard-specific    |

**Best practice:**
Use scene/wizard state whenever possible.

---

## 35️⃣ Leaving / Resetting Scenes

```js
ctx.scene.leave();
```

Force exit:

```js
ctx.scene.reset();
```

Cancel command:

```js
bot.command('cancel', (ctx) => {
  ctx.scene.leave();
  ctx.reply('Cancelled');
});
```

---

## 36️⃣ Persistent Sessions (Important)

Default session storage = **memory** ❌
Crashes = data loss.

### Custom session store example:

```js
bot.use(session({
  store: {
    get: (key) => db[key],
    set: (key, value) => db[key] = value,
    delete: (key) => delete db[key]
  }
}));
```

In real apps, use:

* Redis
* MongoDB
* PostgreSQL

---

## 37️⃣ Rate Limiting (Anti-Spam)

Telegraf doesn’t enforce limits by default.

Typical middleware:

```js
const rateLimit = new Map();

bot.use((ctx, next) => {
  const now = Date.now();
  const last = rateLimit.get(ctx.from.id) || 0;
  if (now - last < 1000) return;
  rateLimit.set(ctx.from.id, now);
  return next();
});
```

Without this → bot abuse guaranteed.

---

## 38️⃣ File Downloading (Telegram Servers)

```js
const fileLink = await ctx.telegram.getFileLink(ctx.message.document.file_id);
```

Then download via HTTP client.

Telegraf **does not auto-download files**.

---

## 39️⃣ Security Rules (Non-Negotiable)

* ❌ Never trust `ctx.message.text`
* ❌ Never expose token
* ❌ Never assume chat is private
* ✔ Validate inputs
* ✔ Check admin permissions in groups
* ✔ Escape Markdown

Admin check:

```js
const admins = await ctx.getChatAdministrators();
```

---

## 40️⃣ Common Production Mistakes

❌ Long logic in handlers
❌ No session persistence
❌ No error handler
❌ Polling on serverless
❌ Blocking operations inside handlers

---

## What comes NEXT (Part 4)

When you type **next**, I’ll cover:

* Deployment (PM2, Docker, Nginx)
* Webhooks properly (HTTPS, SSL)
* Scaling bots
* Telegram limits & pitfalls
* Advanced Telegram API methods
* Real production architecture

👉 **Type `next` to continue**
