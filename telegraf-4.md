Good constraint. I’ll **challenge assumptions**, stay **code-only**, and cover **only** what you asked:

> **Telegram limits & pitfalls**
> **Advanced Telegram Bot API methods via Telegraf**
> No hosting, no PM2, no Docker, no infra talk.

Everything below is **Telegraf v4.x + latest Telegram Bot API behavior**.

---

# 📘 TELEGRAF (npm) — PART 4

## Telegram Limits, Pitfalls & Advanced API (CODE-FOCUSED)

---

## 41️⃣ HARD Telegram Limits (You cannot code around these)

If your bot breaks at scale, it’s usually because you ignored these.

### ⛔ Message limits

| Limit                          | Value           |
| ------------------------------ | --------------- |
| Messages per second (global)   | **~30 msg/sec** |
| Messages per second (per chat) | **1 msg/sec**   |
| Messages per minute (group)    | ~20             |
| Caption length                 | **1024 chars**  |
| Message text length            | **4096 chars**  |

❌ Looping `ctx.reply()` will get your bot **rate-limited or blocked**.

### Correct batching pattern

```js
await ctx.reply('Part 1');
await new Promise(r => setTimeout(r, 1100));
await ctx.reply('Part 2');
```

---

## 42️⃣ File Size Limits (Non-negotiable)

| Type     | Limit |
| -------- | ----- |
| Photo    | 10 MB |
| Video    | 50 MB |
| Document | 2 GB  |
| Voice    | 20 MB |

Telegraf **does not bypass** these.

Always validate:

```js
if (ctx.message.document.file_size > 20 * 1024 * 1024) {
  return ctx.reply('File too large');
}
```

---

## 43️⃣ Markdown Pitfall (Top #1 crash reason)

### ❌ WRONG

```js
ctx.reply(`Hello _${username}_`, { parse_mode: 'MarkdownV2' });
```

This WILL break if username contains special chars.

### ✅ CORRECT (escape)

```js
const escape = (text) =>
  text.replace(/[_*[\]()~`>#+\-=|{}.!]/g, '\\$&');

ctx.reply(
  `Hello _${escape(username)}_`,
  { parse_mode: 'MarkdownV2' }
);
```

**Rule:**
If user input → always escape.

---

## 44️⃣ Update Types You MUST Handle (or ignore intentionally)

Telegram sends many update types devs forget.

```js
bot.on('edited_message', ctx => {});
bot.on('channel_post', ctx => {});
bot.on('my_chat_member', ctx => {});
bot.on('chat_member', ctx => {});
```

If your bot is in groups/channels and you don’t guard these, **logic misfires**.

Example guard:

```js
if (!ctx.message?.text) return;
```

---

## 45️⃣ Advanced `ctx.telegram` API (Power tools)

Most devs never use these — that’s a mistake.

---

### 🔹 Send message to ANY chat

```js
ctx.telegram.sendMessage(chatId, 'Hello');
```

---

### 🔹 Forward messages

```js
ctx.telegram.forwardMessage(
  targetChatId,
  fromChatId,
  messageId
);
```

---

### 🔹 Copy message (no “forwarded” label)

```js
ctx.telegram.copyMessage(
  targetChatId,
  fromChatId,
  messageId
);
```

---

### 🔹 Delete messages

```js
ctx.telegram.deleteMessage(chatId, messageId);
```

⚠ Bot must be admin in groups.

---

### 🔹 Pin message

```js
ctx.telegram.pinChatMessage(chatId, messageId);
```

---

### 🔹 Edit message text (by ID)

```js
ctx.telegram.editMessageText(
  chatId,
  messageId,
  null,
  'Updated text'
);
```

---

## 46️⃣ Message Reactions (NEW Telegram feature)

```js
ctx.telegram.setMessageReaction(
  chatId,
  messageId,
  [{ type: 'emoji', emoji: '🔥' }]
);
```

⚠ Not all clients support reactions yet.

---

## 47️⃣ Chat Permissions (Mute / Restrict Users)

### Restrict user

```js
ctx.telegram.restrictChatMember(chatId, userId, {
  can_send_messages: false
});
```

### Unrestrict

```js
ctx.telegram.restrictChatMember(chatId, userId, {
  can_send_messages: true
});
```

Bot must be admin.

---

## 48️⃣ Ban / Unban Users

```js
ctx.telegram.banChatMember(chatId, userId);
ctx.telegram.unbanChatMember(chatId, userId);
```

**Pitfall:**
Ban ≠ kick forever unless you specify duration.

---

## 49️⃣ Inline Query API (Advanced Bots)

Inline bots respond **without chat**.

```js
bot.on('inline_query', async (ctx) => {
  await ctx.answerInlineQuery([
    {
      type: 'article',
      id: '1',
      title: 'Result',
      input_message_content: {
        message_text: 'Inline response'
      }
    }
  ]);
});
```

Limit:

* Max **50 results**
* Must respond **within 5 seconds**

---

## 50️⃣ Callback Query Pitfall (Button spam bug)

If you don’t answer callback queries fast:

```js
ctx.answerCbQuery();
```

Telegram shows:

> “Loading…”

And users click repeatedly → spam.

---

## 51️⃣ Media Groups (Albums)

Send multiple images/videos together:

```js
ctx.replyWithMediaGroup([
  { type: 'photo', media: 'https://img1.jpg' },
  { type: 'photo', media: 'https://img2.jpg' }
]);
```

Limit: **10 items max**

---

## 52️⃣ Deep Linking (Advanced Start Payload)

```js
bot.start((ctx) => {
  const payload = ctx.startPayload;
  if (payload === 'ref123') {
    ctx.reply('Referral detected');
  }
});
```

Start link:

```
https://t.me/yourbot?start=ref123
```

---

## 53️⃣ Payments API (Telegram Native)

```js
ctx.replyWithInvoice({
  title: 'Premium',
  description: '1 month access',
  payload: 'invoice_payload',
  provider_token: 'PROVIDER_TOKEN',
  currency: 'INR',
  prices: [{ label: 'Premium', amount: 9900 }]
});
```

You must also handle:

```js
bot.on('pre_checkout_query', ctx => ctx.answerPreCheckoutQuery(true));
```

---

## 54️⃣ Web App (Mini Apps) Integration

```js
Markup.button.webApp('Open App', 'https://yourapp.com');
```

Receive data:

```js
bot.on('web_app_data', ctx => {
  console.log(ctx.webAppData.data);
});
```

---

## 55️⃣ Silent Failures (Most dangerous pitfall)

Telegram **silently ignores**:

* Invalid message edits
* Editing old messages
* Invalid callback answers

Always wrap advanced calls:

```js
try {
  await ctx.editMessageText('Updated');
} catch (e) {
  console.error(e);
}
```

---

## 56️⃣ Golden Rules (Don’t argue with these)

* Never trust user input
* Escape Markdown
* Rate limit everything
* Check chat type
* Handle callback queries
* Expect silent failures

---

## What remains (next part, if you want)

If you type **next**, I will continue with **only code-level topics**, such as:

* Bot architecture patterns
* Command routing patterns
* Permission systems
* Large bot code organization
* Testing Telegraf bots

No hosting, no infra — strictly **bot code**.

Just type **next**.
