# `dexster-discord-transcript`

[![npm](https://img.shields.io/npm/dw/dexster-discord-transcript)](http://npmjs.org/package/dexster-discord-transcript)
![GitHub package.json version](https://img.shields.io/github/package-json/v/dexster/dexster-discord-transcript)

A powerful node.js module to generate nice looking HTML transcripts for Discord. Processes discord markdown like **bold**, _italics_, ~~strikethroughs~~, and more. Nicely formats attachments and embeds. Built in XSS protection, preventing users from inserting arbitrary html tags.

This module can format the following:

- Discord flavored markdown
- Embeds
- System messages
  - Join messages
  - Message Pins
  - Boost messages
- Slash commands
- Buttons
- Reactions
- Attachments
  - Images, videos, audio, and generic files
- Replies
- Mentions
- Threads

**This module is designed to work with [discord.js](https://discord.js.org/#/) v14/v15.**

Behind the scenes, this package uses React SSR to generate a static site.

## 📝 Usage

### Example usage using the built in message fetcher.

```js
const discordTranscripts = require('dexster-discord-transcript');
// or (if using typescript) import * as discordTranscripts from 'dexster-discord-transcript';

const channel = message.channel; // or however you get your TextChannel

// Must be awaited
const attachment = await discordTranscripts.createTranscript(channel);

channel.send({
  files: [attachment],
});
```

### Or if you prefer, you can pass in your own messages.

```js
const discordTranscripts = require('dexster-discord-transcript');
// or (if using typescript) import * as discordTranscripts from 'dexster-discord-transcript';

const messages = someWayToGetMessages(); // Must be Collection<string, Message> or Message[]
const channel = someWayToGetChannel(); // Used for ticket name, guild icon, and guild name

// Must be awaited
const attachment = await discordTranscripts.generateFromMessages(messages, channel);

channel.send({
  files: [attachment],
});
```

## ⚙️ Configuration

Both methods of generating a transcript allow for an option object as the last parameter.
**All configuration options are optional!**

### Built in Message Fetcher

```js
const attachment = await discordTranscripts.createTranscript(channel, {
    limit: -1, // Max amount of messages to fetch. `-1` recursively fetches.
    returnType: 'attachment', // Valid options: 'buffer' | 'string' | 'attachment' Default: 'attachment' OR use the enum ExportReturnType
    filename: 'transcript.html', // Only valid with returnType is 'attachment'. Name of attachment.
    saveImages: false, // Download all images and include the image data in the HTML (allows viewing the image even after it has been deleted) (! WILL INCREASE FILE SIZE !)
    footerText: "Exported {number} message{s}", // Change text at footer, don't forget to put {number} to show how much messages got exported, and {s} for plural
    callbacks: {
      // register custom callbacks for the following:
      resolveChannel: (channelId: string) => Awaitable<Channel | null>,
      resolveUser: (userId: string) => Awaitable<User | null>,
      resolveRole: (roleId: string) => Awaitable<Role | null>,
      resolveImageSrc: (
        attachment: APIAttachment,
        message: APIMessage
      ) => Awaitable<string | null | undefined>
    },
    poweredBy: true, // Whether to include the "Powered by" footer
    hydrate: true, // Whether to hydrate the html server-side
    filter: (message) => true // Filter messages, e.g. (message) => !message.author.bot
});
```

### Providing your own messages

```js
const attachment = await discordTranscripts.generateFromMessages(messages, channel, {
  // Same as createTranscript, except no limit or filter
});
```

### Compressing images

If `saveImages` is set to `true`, all images will be downloaded and stored in the file _as-is_. You can optionally enable compression by installing the `sharp` module and setting the following options:

```js
callbacks: {
  resolveImageSrc: new TranscriptImageDownloader()
    .withMaxSize(5120) // 5MB in KB
    .withCompression(40, true) // 40% quality, convert to webp
    .build(),
},
```

Note that, in a more advanced setup, you could store a copy of the files and return an entirely new URL pointing to your own image hosting site by implementing a custom `resolveImageSrc` function.
