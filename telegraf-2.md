Good — continuing **strictly with the latest Telegraf v4.x documentation**, deeper and more technical.
I’ll also **push back on wrong mental models** where needed, because most Telegram bots fail due to misuse, not lack of features.

---

# 📘 **TELEGRAF (npm) – PART 2**

## (Latest v4.x)

**Focus:** Context API, Telegram API, Replies, Keyboards, Buttons

---

## 11️⃣ `ctx.reply()` vs `ctx.telegram.sendMessage()`

Many devs misuse these. They are **not the same abstraction level**.

### ✅ `ctx.reply()` (recommended 90% of the time)

```js
bot.on('text', (ctx) => {
  ctx.reply('Hello');
});
```

✔ Automatically replies to the current chat
✔ Cleaner
✔ Safer

---

### ⚠ `ctx.telegram.sendMessage()` (low-level)

```js
bot.on('text', (ctx) => {
  ctx.telegram.sendMessage(ctx.chat.id, 'Hello');
});
```

Use this only when:

* Sending to **another chat**
* Sending messages **outside current context**

**Wrong assumption:**

> “telegram.sendMessage is more powerful”
> ❌ No. It’s just lower-level.

---

## 12️⃣ Accessing Message, User, Chat Data

### Message

```js
ctx.message.text
ctx.message.from
ctx.message.message_id
```

### User

```js
ctx.from.id
ctx.from.username
ctx.from.first_name
```

### Chat

```js
ctx.chat.id
ctx.chat.type   // private | group | supergroup | channel
```

If you don’t check `ctx.chat.type`, your bot **will break in groups**.

---

## 13️⃣ Reply Options (Formatting, Reply To, Disable Preview)

```js
ctx.reply(
  '*Bold text*\n_Italic text_',
  {
    parse_mode: 'Markdown',
    reply_to_message_id: ctx.message.message_id,
    disable_web_page_preview: true
  }
);
```

### Supported parse modes:

* `Markdown`
* `MarkdownV2` (strict)
* `HTML`

⚠ **MarkdownV2 breaks easily** if you don’t escape characters.

---

## 14️⃣ Sending Media (Images, Video, Audio, Files)

### Photo

```js
ctx.replyWithPhoto(
  'https://example.com/image.jpg',
  { caption: 'Nice photo' }
);
```

### Video

```js
ctx.replyWithVideo(
  { source: './video.mp4' },
  { caption: 'Watch this' }
);
```

### Document

```js
ctx.replyWithDocument(
  { source: './file.pdf' }
);
```

**Reality check:**
Telegraf **does not upload magically** — Telegram size limits still apply.

---

## 15️⃣ Reply Keyboard (Custom Keyboard)

### Basic Reply Keyboard

```js
const { Markup } = require('telegraf');

ctx.reply(
  'Choose option',
  Markup.keyboard([
    ['Yes', 'No'],
    ['Maybe']
  ])
  .resize()
);
```

✔ Appears instead of phone keyboard
✔ Stays until removed

### Remove keyboard

```js
ctx.reply('Keyboard removed', Markup.removeKeyboard());
```

---

## 16️⃣ Inline Keyboard (Buttons under message)

This is what **real bots use**.

```js
ctx.reply(
  'Click button',
  Markup.inlineKeyboard([
    Markup.button.callback('Click me', 'btn_click')
  ])
);
```

Handle action:

```js
bot.action('btn_click', (ctx) => {
  ctx.answerCbQuery();
  ctx.reply('Button clicked');
});
```

⚠ If you forget `answerCbQuery()`, Telegram may show a loading spinner forever.

---

## 17️⃣ URL Buttons

```js
Markup.inlineKeyboard([
  Markup.button.url('Open site', 'https://example.com')
])
```

No callback here — Telegram opens the link directly.

---

## 18️⃣ Editing Messages (Critical for clean UX)

### Edit text

```js
ctx.editMessageText('Updated text');
```

### Edit buttons

```js
ctx.editMessageReplyMarkup(
  Markup.inlineKeyboard([
    Markup.button.callback('New Button', 'new')
  ])
);
```

⚠ Works **only** on messages sent by your bot.

---

## 19️⃣ Answering Callback Queries (VERY IMPORTANT)

If you handle inline buttons, **you must respond**:

```js
bot.action('new', async (ctx) => {
  await ctx.answerCbQuery('Processed!');
});
```

Or silently:

```js
ctx.answerCbQuery();
```

If you don’t:

* UX breaks
* Telegram warns user

---

## 20️⃣ `hears()` vs `on('text')`

### `hears()` (pattern-based)

```js
bot.hears(/hello/i, ctx => ctx.reply('Hi'));
```

### `on('text')` (all text)

```js
bot.on('text', ctx => ctx.reply('Any text'));
```

**Wrong belief:**

> hears is faster
> ❌ No. It’s just filtered.

---

## 21️⃣ Filters (Modern & Correct Way)

```js
const { message } = require('telegraf/filters');

bot.on(message('photo'), ctx => {
  ctx.reply('Photo received');
});
```

This is **preferred** over string-based filters.

---

## 22️⃣ Error Handling (Most bots ignore this)

```js
bot.catch((err, ctx) => {
  console.error(`Error for ${ctx.updateType}`, err);
});
```

Without this, your bot can **crash silently**.

---

## 23️⃣ Environment Variables (Production Reality)

Never hardcode token.

```bash
BOT_TOKEN=123456:ABC
```

```js
const bot = new Telegraf(process.env.BOT_TOKEN);
```

If your token leaks → bot hijacked → irreversible.

---

## 24️⃣ Polling vs Webhooks (Truth)

| Polling              | Webhooks            |
| -------------------- | ------------------- |
| Easy                 | Scalable            |
| Dev only             | Production          |
| Long-running process | Serverless friendly |

**Wrong assumption:**

> Webhooks are optional in production
> ❌ For serious scale, polling is wrong.

---

## What’s coming NEXT (Part 3)

When you type **next**, I will cover:

* Sessions (state management)
* Scenes & Wizards (multi-step flows)
* File uploads & downloads
* Rate limiting
* Security best practices
* Deployment (PM2, Docker, serverless)
* Telegram API edge cases
* Real production patterns

👉 **Type `next` to continue**
