# `dexster-discord-transcript`

[![npm](https://img.shields.io/npm/dw/dexster-discord-transcript)](http://npmjs.org/package/dexster-discord-transcript)
[![npm version](https://img.shields.io/npm/v/dexster-discord-transcript)](http://npmjs.org/package/dexster-discord-transcript)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](LICENSE)
[![discord.js](https://img.shields.io/badge/discord.js-v14%2Fv15-5865F2)](https://discord.js.org)

A robust Node.js module for generating high-fidelity HTML transcripts for Discord. This package specializes in producing transcripts that maintain the exact visual integrity of the Discord interface, supporting modern features and components.

---

## Arabic Documentation | التوثيق بالعربية

تعد هذه المكتبة أداة قوية لمطوري برامج ديسكورد (Discord Bots) لإنشاء سجلات محادثات (Transcripts) بصيغة HTML تتميز بدقة عالية ومحاكاة تامة لواجهة ديسكورد الرسمية.

### المميزات الرئيسية:
- **دقة التصميم:** محاكاة كاملة لواجهة ديسكورد (الوضع الداكن، الخطوط الرسمية، والأبعاد).
- **دعم المكونات الحديثة:** دعم كامل لـ Components V2 (Containers, Sections, Media Galleries).
- **تخصيص كامل:** إمكانية تعديل الألوان، أسماء الملفات، والنصوص التذييلية.
- **الأمان:** حماية مدمجة ضد هجمات XSS لضمان سلامة العرض في المتصفحات.

---

## Key Features

- **High-Fidelity Rendering**: Precise replication of the Discord UI, including dark mode styling, typography, and layout.
- **Markdown Support**: Full processing of Discord flavored markdown (bold, italics, code blocks, etc.).
- **Component V2 Support**: Complete implementation of modern Discord layout components like Containers, Sections, and Media Galleries.
- **Interactive Elements**: Properly styled buttons, action rows, and reactions.
- **Enhanced Mentions**: Accurate rendering of user, role, and channel mentions with integrated role coloring.
- **System Messaging**: Support for system events such as user joins, boosts, and pinned messages.
- **XSS Protection**: Comprehensive sanitization to ensure transcripts are safe for browser execution.
- **Image Persistence**: Capability to embed image data directly within the HTML for offline accessibility.
- **Lightweight Output (Minified by default)**: Generated HTML is automatically minified (no HTML/CSS/JS comments + compact markup) to reduce file size.
- **Hosted Transcripts (Open on the Web)**: Upload transcripts to your own hosting provider and send a URL instead of forcing users to download files.

---

## Installation

```bash
npm install dexster-discord-transcript
# or
pnpm add dexster-discord-transcript
# or
yarn add dexster-discord-transcript
```

---

## Usage

### Basic Implementation

```javascript
const discordTranscripts = require('dexster-discord-transcript');

// For TypeScript:
// import * as discordTranscripts from 'dexster-discord-transcript';

const channel = message.channel; 

const attachment = await discordTranscripts.createTranscript(channel);

channel.send({ files: [attachment] });
```

### Hosted Transcript (Open in browser)

Instead of sending an `.html` file attachment, you can upload the generated transcript to any hosting provider you want
(your own server, Cloudflare R2, S3, a paste service, etc.) and send the **URL**.

```javascript
const discordTranscripts = require('dexster-discord-transcript');

const result = await discordTranscripts.createHostedTranscript(message.channel, {
  // You implement the upload — keep the provider choice fully in your hands.
  upload: async ({ html, filename, password }) => {
    // Example pseudo-code:
    // const url = await myUploader(html, { filename, password })
    // return { url }
    return { url: 'https://your-host.example/transcripts/' + filename };
  },

  // Optional: static password or auto-generated
  // password: 'MySecret123',
  passwordLength: 12,

  // Optional: name
  filename: `transcript-${message.channel.id}.html`,
});

// result = { url, password, filename }
await message.reply(
  `Transcript: ${result.url}\nPassword: ${result.password}`
);
```

### Generating from Message Collections

```javascript
const discordTranscripts = require('dexster-discord-transcript');

const messages = getMessages(); // Collection<string, Message> or Message[]
const channel = getChannel();

const attachment = await discordTranscripts.generateFromMessages(messages, channel);

channel.send({ files: [attachment] });
```

---

## Configuration Options

### `createTranscript(channel, options)`

| Option | Type | Description |
| :--- | :--- | :--- |
| `limit` | `number` | Maximum messages to fetch (-1 for all). |
| `returnType` | `string` | Output format: 'buffer', 'string', or 'attachment'. |
| `filename` | `string` | Name of the output file. |
| `saveImages` | `boolean` | Embed images directly in HTML. |
| `poweredBy` | `boolean` | Toggle the "Powered by" footer visibility. |
| `hydrate` | `boolean` | Enable server-side hydration. |
| `filter` | `function` | Predicate function to filter messages. |

### `createHostedTranscript(channel, options)`

Same options as `createTranscript(..., { returnType: 'string' })` plus:

- **`upload`**: `(payload) => Promise<{ url: string }>` (required)  
  Receives `{ html, filename, password }` and must return `{ url }`.
- **`password`**: `string` (optional) — if omitted, a random password is generated.
- **`passwordLength`**: `number` (optional) — default 12.

---

## Advanced: Image Compression

Integration with `sharp` allows for automated image optimization when using `saveImages`:

```javascript
const { TranscriptImageDownloader } = require('dexster-discord-transcript');

const attachment = await discordTranscripts.createTranscript(channel, {
  saveImages: true,
  callbacks: {
    resolveImageSrc: new TranscriptImageDownloader()
      .withMaxSize(5120)            // Maximum size in KB
      .withCompression(40, true)    // 40% quality, convert to WebP
      .build(),
  },
});
```

---

## Technical Specifications

- **Compatibility**: discord.js v14 & v15.
- **Engine**: React SSR for server-side HTML generation.
- **Runtime**: Node.js 18.x or higher.
- **License**: Apache-2.0.

---

## Author

Developed and maintained by **Ebrahim (1Dexster1)**.
[GitHub Profile](https://github.com/1Dexster1)
