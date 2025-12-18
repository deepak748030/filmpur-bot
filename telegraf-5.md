Good. Now we’re deep enough that **small design mistakes become expensive bugs**.
This part is **pure code architecture & patterns for Telegraf bots** — no hosting, no infra, no fluff.

Everything is **latest Telegraf v4.x–compatible**.

---

# 📘 TELEGRAF (npm) — PART 5

## Bot Architecture, Command Routing, Permissions, Large-Bot Patterns

---

## 57️⃣ Why most Telegram bots become unmaintainable

Common wrong assumption:

> “It’s just a bot, I’ll keep everything in one file”

That works until:

* 20+ commands
* multiple roles (admin/user)
* sessions + scenes
* callbacks + inline queries

**Result:** unreadable spaghetti.

---

## 58️⃣ Minimal scalable folder structure (code-only)

 constants.js


This is not “enterprise overkill” — it’s **necessary**.

---

## 59️⃣ Central bot initialization (`bot.js`)

```js
const { Telegraf, session } = require('telegraf');
const stage = require('./scenes');

const bot = new Telegraf(process.env.BOT_TOKEN);

bot.use(session());
bot.use(stage.middleware());

module.exports = bot;
```

No logic here — only wiring.

---

## 60️⃣ Command routing pattern (clean & testable)

### `commands/start.js`

```js
module.exports = (bot) => {
  bot.start((ctx) => {
    ctx.reply('Welcome');
  });
};
```

### `commands/help.js`

```js
module.exports = (bot) => {
  bot.command('help', (ctx) => {
    ctx.reply('/start\n/help\n/settings');
  });
};
```

### Load all commands

```js
// index.js
const bot = require('./bot');

require('./commands/start')(bot);
require('./commands/help')(bot);
require('./commands/admin')(bot);

bot.launch();
```

**Why this matters:**
Each command becomes isolated, readable, testable.

---

## 61️⃣ Action / Callback routing

### `actions/buttons.js`

```js
module.exports = (bot) => {
  bot.action('approve', async (ctx) => {
    await ctx.answerCbQuery();
    ctx.editMessageText('Approved');
  });

  bot.action('reject', async (ctx) => {
    await ctx.answerCbQuery();
    ctx.editMessageText('Rejected');
  });
};
```

Register once, not inline.

---

## 62️⃣ Permission system (non-negotiable for real bots)

### Admin guard middleware

```js
const isAdmin = async (ctx, next) => {
  const admins = await ctx.getChatAdministrators();
  const adminIds = admins.map(a => a.user.id);

  if (!adminIds.includes(ctx.from.id)) {
    return ctx.reply('Admins only');
  }

  return next();
};
```

Usage:

```js
bot.command('ban', isAdmin, (ctx) => {
  // admin-only logic
});
```

**Mistake to avoid:**
Checking username instead of user ID ❌

---

## 63️⃣ Role-based permission pattern

```js
const roles = {
  ADMIN: 'admin',
  USER: 'user'
};

const requireRole = (role) => (ctx, next) => {
  if (ctx.session.role !== role) {
    return ctx.reply('Access denied');
  }
  return next();
};
```

Usage:

```js
bot.command('stats', requireRole(roles.ADMIN), handler);
```

---

## 64️⃣ Scene-first architecture (important insight)

❌ Command-first bots break easily
✅ Scene-first bots scale better

Pattern:

* Commands only **enter scenes**
* Scenes handle logic

```js
bot.command('feedback', ctx => ctx.scene.enter('feedback'));
```

This prevents state chaos.

---

## 65️⃣ Service layer (separate business logic)

### `services/user.service.js`

```js
module.exports.createUser = async (data) => {
  // db logic
};

module.exports.findUser = async (id) => {
  // db logic
};
```

Usage in bot:

```js
const userService = require('../services/user.service');

bot.on('text', async (ctx) => {
  await userService.createUser({
    telegramId: ctx.from.id
  });
});
```

**Golden rule:**
Bot files ≠ business logic.

---

## 66️⃣ Guard against duplicate updates

Telegram **can resend updates**.

```js
const processed = new Set();

bot.use((ctx, next) => {
  const id = ctx.update.update_id;
  if (processed.has(id)) return;
  processed.add(id);
  return next();
});
```

Without this, double execution happens.

---

## 67️⃣ Idempotent handlers (critical concept)

Bad:

```js
createOrder();
sendConfirmation();
```

Good:

```js
if (orderExists) return;
createOrder();
sendConfirmation();
```

Telegram retries → your code must survive retries.

---

## 68️⃣ Global error boundary (MANDATORY)

```js
bot.catch((err, ctx) => {
  console.error('Bot error:', err);
  ctx.reply?.('Something went wrong');
});
```

Without this, failures are silent.

---

## 69️⃣ Testing Telegraf handlers (yes, you should)

Mock context:

```js
const ctx = {
  from: { id: 1 },
  message: { text: 'hi' },
  reply: jest.fn()
};

await handler(ctx);

expect(ctx.reply).toHaveBeenCalled();
```

Handlers should be **pure functions where possible**.

---

## 70️⃣ Performance rules (code-level)

* ❌ No heavy loops inside handlers
* ❌ No blocking I/O
* ✔ Use async/await
* ✔ Cache admin lists
* ✔ Batch messages

Telegram bots are **event-driven**, not servers.

---

## 71️⃣ When to split bots (advanced insight)

If you have:

* user bot
* admin bot
* notification bot

**Split tokens, split responsibilities.**
One giant bot is fragile.

---

## 72️⃣ Final mental model (important)

Telegram bots are:

* Stateless by default
* Retry-prone
* Rate-limited
* Event-driven

Your code must be:

* Defensive
* Idempotent
* Modular
* State-aware

---

### What’s left (final part)

If you type **next**, I’ll finish with:

* Testing strategies in depth
* Mocking Telegram updates
* Edge-case patterns
* Final best-practice checklist

No repeats, no infra, strictly **bot code**.
