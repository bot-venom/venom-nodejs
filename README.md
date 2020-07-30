<div align="center">
<img src="https://user-images.githubusercontent.com/69011198/88970782-641a0200-d289-11ea-960d-2fed84a35738.png" width="128" height="128"/>

# venom-bot-nodejs

> Venom is a high-performance system developed with JavaScript to create a bot for WhatsApp, support for creating any interaction, such as customer service, media sending, sentence recognition based on artificial intelligence and all types of design architecture for WhatsApp.

[![npm version](https://img.shields.io/npm/v/@open-wa/wa-automate.svg?color=green)](https://www.npmjs.com/package/@open-wa/wa-automate)
![node](https://img.shields.io/node/v/@open-wa/wa-automate)
[![Downloads](https://img.shields.io/npm/dm/@open-wa/wa-automate.svg)](https://www.npmjs.com/package/@open-wa/wa-automate)

</div>

## Installation

```bash
> npm i --save venom-bot
```

## Usage

```javascript
// Supports ES6
// import { create, Whatsapp } from 'venom-bot';
const venom = require('venom-bot')

venom.create().then(client => start(client))

function start (client) {
  client.onMessage(message => {
    if (message.body === 'Hi') {
      client.sendText(message.from, 'Welcome Venom 🕷')
    }
  })
}
```

![ULsDl6SQ10](https://user-images.githubusercontent.com/40338524/88978275-c547d280-d295-11ea-8e82-41d496517472.gif)

###### After executing `create()` function, **Venom** will create an instance of WA web. If you are not logged in, it will print a QR code in the terminal. Scan it with your phone and you are ready to go!

###### **Venom** will remember the session so there is no need to authenticate every time

###### Multiples sessions can be created at the same time by pasing a session name to create() function:

```javascript
// Init sales whatsapp bot
venom.create('sales').then((salesClient) => {...});

// Init support whatsapp bot
venom.create('support').then((supportClient) => {...});
```

## Optional create params

Venom create() method third parameter can have the following optional parameters:

```javascript
create('sessionName', qrCallback, statusFind, {
  headless: true, // Headless chrome
  devtools: false, // Open devtools by default
  useChrome: true, // If false will use Chromium instance
  debug: false, // Opens a debug session
  logQR: true, // Logs QR automatically in terminal
  browserArgs: [''], // Parameters to be added into the chrome browser instance
  refreshQR: 15000, // Will refresh QR every 15 seconds, 0 will load QR once. Default is 30 seconds
  autoClose: 60000, // Will auto close automatically if not synced, 'false' won't auto close. Default is 60 seconds (#Important!!! Will automatically set 'refreshQR' to 1000#)
  disableSpins: true // Will disable Spinnies animation, useful for containers (docker) for a better log
})
```

### Functions list

| Function                                                               | Implemented |
| ---------------------------------------------------------------------- | ----------- |
| Receive message                                                        | ✅          |
| Automatic QR Refresh                                                   | ✅          |
| Send text                                                              | ✅          |
| Get contacts                                                           | ✅          |
| Get chats                                                              | ✅          |
| Get groups                                                             | ✅          |
| Get group members                                                      | ✅          |
| Send contact                                                           | ✅          |
| Send Images (image)                                                    | ✅          |
| Send media (audio, doc)                                                | ✅          |
| Send media (video)                                                     | ✅          |
| Send stickers                                                          | ✅          |
| Decrypt media (image, audio, doc)                                      | ✅          |
| Automatic QR Refresh                                                   | ✅          |
| Multiple Sessions                                                      | ✅          |
| Send Location                                                          | ✅          |
| 🕸🕸 And much more                                                     | ✅          |

Not every available function is listed, for further look, every function available can be found in [here](https://github.com/orkestral/venom/tree/master/src/api/layers) and [here](https://github.com/orkestral/venom/tree/master/src/lib/wapi/functions)

## Callback Status Session

Gets the return if the session is isLogged or if it is notLogged

```javascript
create('sessionName', qrCallback, statusFind => {
  console.log(statusFind)
})
  .then(client => {
    start(client)
  })
  .catch(erro => console.log(erro))
```

## Exporting QR Code

An event is emitted every time the QR code is received by the system. You can grab hold of this event emitter by importing `ev`

```javascript
const fs = require('fs')

// Second create() parameter is the QR callback
venom.create('sessionMarketing', (base64Qr, asciiQR) => {
  // To log the QR in the terminal
  console.log(asciiQR)

  // To write it somewhere else in a file
  exportQR(base64Qr, 'marketing-qr.png')
})

// Writes QR in specified path
function exportQR (qrCode, path) {
  qrCode = qrCode.replace('data:image/png;base64,', '')
  const imageBuffer = Buffer.from(qrCode, 'base64')

  // Creates 'marketing-qr.png' file
  fs.writeFileSync(path, imageBuffer)
}
```

## Kill the session

As of v1.6.6^ you can now kill the session when required. Best practice is to manage trycatch-es yourself and kill the client on catch.

```javascript
// Catch ctrl+C
process.on('SIGINT', function() {
  client.close();
});

// Try-catch close
try {
   ...
} catch (error) {
   client.close();
}
```

## Auto closing unsynced sessions

The auto close is enabled by default and the timeout is setted to 60 sec. Receives the time in milliseconds to countdown until paired.

Important with autoClose enabled the "refreshQR" parameter is changed to 1000 (1 sec).
Use "autoClose: false" to disable auto closing.

```javascript
    headless: false,
    devtools: false,
    useChrome: true,
    debug: false,
    logQR: true,
    refreshQR: 15000,
    autoClose: 6000,
    disableSpins: true,
```

## Force Refocus and reacting to state

When a user starts using _WhatsApp Web_ in a different browser, **venom** will be left on a screen prompting you to click 'Use here'. Now you can now force the client to press 'Use here' everytime the state has changed to 'CONFLICT'. onStateChanged results in 'UNPAIRED', 'CONNECTED' or 'CONFLICT';

```javascript
// In case of being logged out of whatsapp web
// Force it to keep the current session
// State change
client.onStateChange(state => {
  console.log(state)
  const conflits = [
    venom.SocketState.CONFLICT,
    venom.SocketState.UNPAIRED,
    venom.SocketState.UNLAUNCHED
  ]
  if (conflits.includes(state)) {
    client.useHere()
  }
})
```

## Downloading Files

Puppeteer takes care of the file downloading. The decryption is being done as fast as possible (outruns native methods). Supports big files!

```javascript
const venom = require('venom-bot');
const mime = require('mime-types');
const fs = require('fs');

venom.create().then((client) => start(client));

function start(client) {
  client.onMessage((message) => {
    if (message.body === 'Hi') {
      client.sendText(message.from, 'Welcome Venom 🕷');
    }

   if (message.isMedia === true) {
     const buffer = await client.decryptFile(message);
     // At this point you can do whatever you want with the buffer
     // Most likely you want to write it into a file
     const fileName = `some-file-name.${mime.extension(message.mimetype)}`;
     await fs.writeFile(fileName, buffer, (err) => {
     //...
   });
  }

  });
}
```

## Chatting

Here, chatId could be ```<phoneNumber>@c.us``` or ```<phoneNumber>-<groupId>@c.us```

```javascript
//Automatically sends a link with the auto generated link preview. You can also add a custom message to be added.
await client.sendLinkPreview("000000000000@c.us", "https://www.youtube.com/watch?v=V1bFr2SWP1I", "Link title");

// Send basic text
await client.sendText(chatId, '👋 Hello from venom!');

// Send image
await client.sendImage(
  chatId,
  'path/to/img.jpg',
  'image-name.jpg',
  'Caption text'
);

// Send @tagged message
await client.sendMentioned(chatId, 'Hello @5218113130740 and @5218243160777!', [
  '5218113130740',
  '5218243160777',
]);

// Reply to a message
await client.reply(chatId, 'This is a reply!', message.id.toString());

// Reply to a message with mention
await client.reply(
  chatId,
  'Hello @5218113130740 and @5218243160777! This is a reply with mention!',
  message.id.toString(),
  ['5218113130740', '5218243160777']
);

// Send file (venom will take care of mime types, just need the path)
await client.sendFile(chatId, 'path/to/file.pdf', 'cv.pdf', 'Curriculum');

// Send gif
await client.sendVideoAsGif(
  chatId,
  'path/to/video.mp4',
  'video.gif',
  'Gif image file'
);

// Send contact
// contactId: 52155334634@c.us
await client.sendContact(chatId, contactId);

// Forwards messages
await client.forwardMessages(chatId, [message.id.toString()], true);

//Generates sticker from the provided animated gif image and sends it (Send image as animated sticker)
//image path imageBase64 A valid gif image is required. You can also send via http/https (http://www.website.com/img.gif)
await client.sendImageAsStickerGif("000000000000@c.us", './image.gif');

//Generates sticker from given image and sends it (Send Image As Sticker)
// image path imageBase64 A valid png, jpg and webp image is required. You can also send via http/https (http://www.website.com/img.jpg)
await client.sendImageAsSticker("000000000000@c.us", './image.jpg');

// Send location
await client.sendLocation(
  chatId,
  25.6801987,
  -100.4060626,
  'Some address, Washington DC',
  'Subtitle'
);

// Send seen ✔️✔️
await client.sendSeen(chatId);

// Start typing...
await client.startTyping(chatId);

// Stop typing
await client.stopTyping(chatId);

// Set chat state (0: Typing, 1: Recording, 2: Paused)
await client.setChatState(chatId, 0 | 1 | 2);
```

## Retrieving Data

```javascript
// Calls your list of blocked contacts (returns an array)
const getBlockList = await client.getBlockList();

// Retrieve contacts
const contacts = await client.getAllContacts();

// Retrieve all messages in chat
const allMessages = await client.loadAndGetAllMessagesInChat(chatId);

// Retrieve contact status
const status = await client.getStatus(contactId);

// Retrieve user profile
const user = await client.getNumberProfile(contactId);

// Retrieve all unread message
const messages = await client.getAllUnreadMessages();

// Retrieve all chats
const chats = await client.getAllChats();

// Retrieve all groups
const chats = await client.getAllGroups();

// Retrieve profile fic (as url)
const url = await client.getProfilePicFromServer(chatId);

// Retrieve chat/conversation
const chat = await client.getChat(chatId);
```

## Group Functions

```javascript
// groupId or chatId: leaveGroup 52123123-323235@g.us

// Leave group
await client.leaveGroup(groupId);

// Get group members
await client.getGroupMembers(groupId);

// Get group members ids
await client.getGroupMembersIds(groupId);

// Generate group invite url link
await client.getGroupInviteLink(groupId);

// Create group (title, participants to add)
await client.createGroup('Group name', ['123123@c.us', '45456456@c.us']);

// Remove participant
await client.removeParticipant(groupId, '123123@c.us');

// Add participant
await client.addParticipant(groupId, '123123@c.us');

// Promote participant (Give admin privileges)
await client.promoteParticipant(groupId, '123123@c.us');

// Demote particiapnt (Revoke admin privileges)
await client.demoteParticipant(groupId, '123123@c.us');

// Get group admins
await client.getGroupAdmins(groupId);

// Return the group status, jid, description from it's invite link
await client.getGroupInfoFromInviteLink(InviteCode);

// Join a group using the group invite code
await client.joinGroup(InviteCode);
```

## Profile Functions

```javascript
// Set client status
await client.setProfileStatus('On vacations! ✈️');

// Set client profile name
await client.setProfileName('Venom bot');

// Set client profile photo
await client.setProfilePic('path/to/image.jpg');
```

## Device Functions

```javascript
//Delete the Service Worker
await client.killServiceWorker();

//Load the service again
await client.restartService();

// Get device info
await client.getHostDevice();

// Get connection state
await client.getConnectionState();

// Get battery level
await client.getBatteryLevel();

// Is connected
await client.isConnected();

// Get whatsapp web version
await client.getWAVersion();
```

## Events
```javascript
// Listen to messages
client.onMessage(message => {
  ...
})

// Listen to state changes
client.onStateChange(state => {
  ...
});

// Listen to ack's
client.onAck(ack => {
  ...
});

// Listen to live location
// chatId: 'phone@c.us'
client.onLiveLocation(chatId, (liveLocation) => {
  ...
});

// chatId looks like this: '5518156745634-1516512045@g.us'
// Event interface is in here: https://github.com/s2click/venom/blob/master/src/api/model/participant-event.ts
client.onParticipantsChanged(chatId, (event) => {
  ...
});

// Listen when client has been added to a group
client.onAddedToGroup(chatEvent => {
  ...
});
```

## Other

```javascript
//Change the theme
//string types "dark" or "light"
await client.setTheme(types);

//Receive the current theme
//returns string light or dark
await client.getTheme();

// Delete chat
await client.deleteChat(chatId);

// Clear chat messages
await client.clearChat(chatId);

// Delete message (last parameter: delete only locally)
await client.deleteMessage(chatId, message.id.toString(), false);

// Mark chat as not seen (returns true if it works)
await client.markUnseenMessage('0000000@c.us');

//blocks a user (returns true if it works)
await client.blockContact('0000000@c.us');

//unlocks contacts (returns true if it works)
await client.unblockContact('0000000@c.us');

// Retrieve a number profile / check if contact is a valid whatsapp number
const profile = await client.getNumberProfile('0000000@c.us');
```

## Multiple sessions
If you need to run multiple sessions at once just pass a session name to create() method, not use hyphen for name of sessions.

```javascript
async () => {
  const marketingClient = await venom.create('marketing');
  const salesClient = await venom.create('sales');
  const supportClient = await venom.create('support');
};
```

## Development

Building venom is really simple altough it contians 3 main projects inside

###### Wapi project

```javascript
npm run build:wapi
```

###### Middleeware

```javascript
npm run build:middleware
npm run build:jsQR
```

###### Venom
```javascript
npm run build:venom
```

To build the entire project just run

```javascript
npm run build
```

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
