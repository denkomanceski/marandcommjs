# MarandCommJS
This is a small js library that is used to interact with the Marand communication platform.

* Required Dependencies: socket.io


*Always include socket.io before including this library*


Example of how to use it is under examples folder.

#  API Reference

- [`MarandCommJS`](#xmppclient)
  - [`new CommClient()`](#clientclientoptions)
  - [`CommClient.createClient(auth)`](#createclientoptions)
  - [`Client` Methods](#client-methods)
    - [`client.on(event, handler)`](#clientonevent-group-handler)
    - [`client.connect([opts])`](#clientconnectops)
    - [`client.disconnect()`](#clientdisconnect)
    - [`client.sendMessage(opts)`](#clientsendmessageopts)
    - [`client.queryContent(contentString)`](#clientqueryContent)
    - [`client.disconnect()`](#clientdisconnect)
    - [`client.getRoster([cb])`](#clientgetRoster)
    - [`client.getVCard(jid, [cb])`](#clientgetVCard)
    - [`client.enableCarbons()`](#clientenableCarbons)
    - [`client.sendPresence()`](#clientsendPresence)
    - [`client.searchHistory(opts, [cb])`](#clientsearchHistory)
    - [`client.joinRoom(roomJid, nickname, config, [cb])`](#clientjoinRoom)
    - [`client.addBookmark(config)`](#clientaddBookmark)
    - [`client.sendMessage(message)`](#clientsendMessage)
    - [`client.configureRoom(roomJid, config, [cb])`](#clientconfigureRoom)
    - [`client.setRoomAffiliation(roomJid, memberJid, memberType, emp, [cb])`](#clientsetRoomAffiliation)
    - [`client.directInvite(roomJid, conf)`](#clientdirectInvite)
    - [`client.destroyRoom(roomJid, conf, [cb])`](#clientdestroyRoom)
    - [`client.removeBookmark(roomJid)`](#clientremoveBookmark)
    - [`client.getDiscoItems(domain, '', [cb])`](#clientgetDiscoItems)
    - [`client.getBookmarks([cb])`](#clientgetBookmarks)
    - [`client.setBookmarks(bookmarks)`](#clientsetBookmarks)
    - [`client.leaveRoom(roomJid, [cb])`](#clientleaveRoom)
- [Events](#events)

    - [`available`](#available)
    - [`carbon:sent`](#carbonsent)
    - [`chat:state`](#chatstate)
    - [`chat`](#chat)
    - [`connected`](#connected)
    - [`disconnected`](#disconnected)
    - [`groupchat`](#groupchat)
    - [`message`](#message)
    - [`muc:destroyed`](#mucdestroyed)
    - [`muc:invite`](#mucinvite)
    - [`presence`](#presence)
    - [`session:error`](#sessionerror)
    - [`session:started`](#sessionstarted)
    - [`queryMetadata`](#queryMetadata)



## `XMPP.Client`

### `new Client(options)`

Creates a new XMPP client instance.

*Instantiating a client via this function will not load any built-in plugins.*

- `options` - An object with the client configuration as described in [client options](#client-options).

```javascript
var XMPP = require('stanza.io');
var client = new XMPP.Client({
    jid: 'test@example.com',
    password: 'hunter2'
});
````

### `createClient(options)`

This is the method to for creating a client instance. It returns the client instance.

- `options` - An object with the client configuration as described in [client options](#client-options).

```javascript
var marandComm = new MarandComm();
var client = marandComm.createClient({
    jid: 'test@example.com',
    password: 'hunter2'
});
```

### `Client` Options

When creating a client instance, the following settings will configure its behaviour:

- `jid` - (required) the requested bare JID for the client.
- `password` - password for the jid.


### `Client` Methods


#### `client.on(event, handler)`
Listen for an event.

Example:
```js

client.on('chat', chat => {
  // chat event handler
});

```
#### `client.connect()`
Connected should be called after setting up a client with `createClient` and registering the event listeners
#### `client.disconnect()`
You should destroy the object and remove the listeners to prevent memory leaks.
#### `client.sendMessage(opts)`
Send a message.

Example:
```js

client.sendMessage({
  to: jid,
  body: body,
  requestReceipt: true,
  type: 'groupchat'
});

```
> "to" attribute needs to contains the jid of the user that you want to send the message to.<br>
> "body" attribute needs to contain the content from the message.<br>
> "requestReceipt" flag is set when you want to receive a notification when the message is delievered to the receipient.
> "type" can be 'groupchat' or 'chat'. If the jid is a from a room, you need to use 'groupchat', otherwise 'chat'


#### `client.queryContent(content)`
Query for tagged content messages. The messages are tagged if they contain a #hashtag

Example:
```js
client.queryContent('medrock')
```
#### `client.sendPresence(opts)`
Send presence to the server.
```js
client.on('session:started', () => {
    client.getRoster(() => {
        client.sendPresence();
    })
});
```
Note the `session:started`. You'll need to send your presence immediately to get presence from rosters.

#### Roster Management
##### `client.getRoster([cb])`
Fetches the jids from the roster for the current user. (Roster = friendlist)

Example:
```js
client.getRoster((err, data) => {
    var roster = data.roster.items;
})
```
#### Service Discovery
##### `client.getDiscoItems(jid, node, [cb])`

Fetch the jids of all available rooms

Example:
```js
client.getDiscoItems('conference.xxx.yyy.zz','', (err, data) => {
            //Get list of available rooms here
}
```

##### `client.configureRoom(room, form, [cb])`
Example:
```js
client.configureRoom('conference.xxx.yyy.zz', fields, (err, data) => {
    //Configure the room with the specified jid.
}
```
> The fields array contains the configuration for the room.


Example for public room fields: <br>

 ```js
 {name: 'FORM_TYPE', value: 'http://jabber.org/protocol/muc#roomconfig'},
 {name: 'muc#roomconfig_persistentroom', value: '1'} // The room is persistent (exists even there isnt any online members)
 ```
Example for private room fields:

```js
{name: 'muc#roomconfig_publicroom', value: '0'}, // 0 means private, 1 means public
{name: 'muc#roomconfig_membersonly', value: '1'}, // If only the members are able to see the room
{name: 'muc#roomconfig_allowinvites', value: '1'} // If it is allowed to send invites for entering in the room.
```


##### `client.destroyRoom(room, opts, [cb])`
> Destroy an existing room.

Example:

```js
client.destroyRoom(room.jid, {
    reason: 'Room destroyed by admin'
}
```
##### `client.directInvite(room, to)`
> Invite a person to the room.

Example:

```js
client.directInvite(room.jid, {
    {to: 'xyz@comm.marand.si'}
}

```
The first parameter is the jid from the room, the second parameter is an objects that contains "to" property which specifies the jid of the user who needs to be invited

##### `client.joinRoom(room, nick, opts)`

Request to join a Multi-User Chat room.

- `room` - The bare JID of the room to join
- `nick` - The requested nickname to use in the room. *Note:* You are not guaranteed to be assigned this nick.
- `options` - Additional presence information to attach to the join request, in particular:
  - `joinMuc` - A dictionary of options for how to join the room:
    - `password` - Optional password for entering the room
    - `history` - May be `true` to receive the room's default history replay, or may specify `maxstanzas`, `seconds`, or `since` to specify a history replay range in terms of number of messages, a given time range, or since a particular time.

Example:
```js
client.joinRoom('room@muc.example.com', 'User', {
    status: 'This will be my status in the MUC',
    joinMuc: {
        password: 'hunter2',
        history: {
            maxstanzas: 20 //Number of messages to fetch from history archive
        }
    }
});
```

##### `client.leaveRoom(room, nick, opts)`
Example:
```js
 client.leaveRoom(roomJid, myJid);
```
> The first param is the jid from the room that you want to leave, the second is your client jid

##### `client.setRoomAffiliation(room, jid, affiliation, reason, [cb])`
Example:
```js
client.setRoomAffiliation(roomJid, memberJid, 'member', '', (err, data)=> {

});
```
> Here you can specify the affiliation for the member inside the room.
A member with `memberJid` can be a 'member', 'owner' or 'admin' of a room with `roomJid`

##### `client.addBookmark(bookmark)`
```js
client.addBookmark({
    jid: roomJid,
    autoJoin: autojoin || true,
    name: roomJid,
    nick: me.name
})
```
> Bookmark is a small storage where you can set some configuration for the room. Than, you are able to fetch that bookmark using `getBookmarks()`

##### `client.getBookmarks([cb])`
> Get the bookmarks for the client.
The bookmarks are stored in `data.privateStorage.bookmarks` in the callback object
Example:


```js
client.getBookmarks((err, data)=> {
    if (data.privateStorage.bookmarks.conferences) {
        var conferences = data.privateStorage.bookmarks.conferences;
})
```
##### `client.removeBookmark(jid, [cb])`

> Remove bookmark for the jid.


##### `client.setBookmarks(opts)`
Set bundle of bookmarks at once.

Example:
```js
client.setBookmarks({conferences: [ARRAY_OF_BOOKMARKS]})
```
#### Message Syncing
##### `client.enableCarbons()`
Enable carbon messages. This is useful if you want to receive a copy of a message that you sent from one device, and receive on all other logged devices.

Example:
```js
client.on('session:started', () => {
    client.sendPresence();
    client.enableCarbons();
});
```

##### `client.searchHistory(opts, [cb])`
Fetch chat history for the specified jid. By default, you will receive all the messages.
Optionally you can pass an object to get max number of messages.

Example:
```js
client.searchHistory({
    with: jid,
    rsm: {max: 50, before: true},
    complete: false
})
```
The before property can also be a timestamp which you receive from a a previous history search. This is used for pagination

#### Other
##### `client.getVCard(jid, [cb])`
Gets a vCard for the user with the specified jid. It can contain phone numbers, user photo..

Example:
```js
client.getVCard(userJid, (err, vcard) => {
    vcard = vcard.vCardTemp;
    //Do something with the vcard..
})
```

## Events
### carbon:sent
This event is called when a carbon is sent. For example a carbon is sent, if the user sends a message to another user from a different device, so all of his online devices will be in sync.
### chat
A 1 on 1 chat message is received here. Room messages are coming in the `message` event.
It contains all the details for the message. (jid, body, type..)

### chat:state
This event is fired when the user you are chatting with is sending events for the current state of the chat. It can be "typing.." or "stoptyping.."

### connected
This event is fired on successful connection to the server.

### disconnected
This event is fired when you are disconnected from the server.

### message
Here you receive the all the messages including the room messages.

Example:
```js
client.on('message', (msg) => {
        if (msg.type === 'groupchat' && msg.body) {
            //Do something with groupchat message
        }
    });
})
```
### muc:invite
This event is fired when someone invites you to join a room. The callback contains the details of the room, the reason..
### presence
You receive the presences from all the users (Online, offline, away..)

**In order to get presence, you have to send your presence first when session starts.**

Example:
```javascript
client.on('session:started', () => {
        client.sendPresence();
});

client.on('presence', (presence) => {
  /*
  {   lang:"en",
      id:"12345",
      to: JID,
      from: JID,
      priority:0,
      type:"available" // Type
    }
  */
});
```
### session:started
This event is fired when the session is started. Here you should fetch the roster, vcards, bookmarks, and most important is call `client.sendPresence()` to register and start receiving presence changes
### queryMetadata
This event is fired after you query for tagged messages. Currently the messages are tagged if they contain a hashtag inside their body ('Hello #MedRock rocks'). You receive the full info for the message here.
The messages are fetched with `client.queryContent(content);`