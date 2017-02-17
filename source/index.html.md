---
title: protoo

language_tabs:
  - javascript

toc_footers:
  - <a href='https://github.com/ibc/protoo'>Source Code at GitHub</a>

search: true
---

# protoo

**protoo** is a minimalist and extensible Node.js signaling framework for Real-Time Communication applications.

It provides both a server and a client Node.js modules to build multi-party Real-Time applications. Its primary purpose is to provide Web applications with the ability to easily add group chat, presence and multimedia capabilities.

**protoo** defines a signaling protocol based on JSON requests and responses. It is up to the application to define and extend the signaling protocol and the content of requests and responses in order to accomplish the desired feature set.


# protoo-server

The **protoo** server side Node.js module.


## Installation

```bash
$ npm install --save protoo-server
```

Install the [protoo-server](https://www.npmjs.com/package/protoo-server) NPM package into your Node.js server side application.


## API

```javascript
const protooServer = require('protoo-server');
```


### Room

```javascript
let room = new protooServer.Room();
```

A `Room` represents a multi-party communication context.


#### `peers`

Returns an array with all the `Peer` instances in the room.


#### `closed`

Boolean indicating whether the room is closed.


#### `createPeer(peerId, transport)`

```javascript
let peer = room.createPeer('alice', transport);
```

Creates a peer within this room. It returns the `Peer` instance.

Parameter    | Description
------------ | ------------------------------
peerId       | Unique string identifier for the peer.
transport    | A `WebSocketTransport` instance.


#### `spread(method, data, excluded)`

```javascript
room.spread('notification', data);
```

Send a request to all the peers in the room.

Parameter    | Description
------------ | ------------------------------
method       | Request method string.
data         | Request data object.
excluded     | Optional array of `Peer` instances or `peerId` string values who won't receive the request.


#### `close()`

Close the room. All the peers within this room will be disconnected.


### WebSocketServer

```javascript
let options =
{
  maxReceivedFrameSize     : 960000, // 960 KBytes.
  maxReceivedMessageSize   : 960000,
  fragmentOutgoingMessages : true,
  fragmentationThreshold   : 960000
};

let server = new WebSocketServer.Room(httpServer, options);
```

A `WebSocketServer` listens for WebSocket connections from clients.

Parameter    | Description
------------ | ------------------------------
httpServer   | A Node.js `http.Server` or `https.Server` object.
options      | Options for [websocket.WebSocketServer](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketServer.md#server-config-options).

<aside class='success'>
It's up to the application to set the <code>http.Server</code> or <code>https.Server</code> and call <code>listen()</code> on it.
<br><br>
In this way, the <code>http.Server</code> or <code>https.Server</code> can also be used for pure HTTP purposes by sharing it with <a href='http://expressjs.com/'>Express</a> or any other Node.js HTTP framework.
</aside>


#### `stop()`

Unmounts the WebSocket server so no more WebSocket connections will take place.

<aside class='notice'>
When <code>stop()</code> is called, the underlying Node.js <code>http.Server</code> or <code>https.Server</code> is not closed.
</aside>


#### `on('connectionrequest', listener)`

```javascript
server.on('connectionrequest', (info, accept, reject) =>
{
  // The app inspects the `info` object and decides whether to accept the
  // connection or not.  
  if (something)
  {
    let transport = accept();

    // The app chooses a `peerId` and creates a peer within a specific room.
    let peer = room.createPeer('bob', transport);
  }
  else
  {
    reject(403, 'Not Allowed');
  }
});
```

Event fired when a WebSocket client attempts to connect to the WebSocket server. The `listener` function is called with the following parameters:

Parameter    | Description
------------ | ------------------------------
info         | An object with information about the connection attempt.
accept       | Function to be called by the application if the connection must be accepted.
reject       | Function to be called by the application if the connection must be rejected.

The `info` object has the following fields:

Field        | Description
------------ | ------------------------------
request      | The Node.js `http.IncomingMessage` object representing the received HTTP request during the Websocket handshake.
origin       | The value of the `Origin` header in the HTTP request.
socket       | The Node.js `net.Socket` object.

The `accept` function returns a `WebSocketTransport` instance.

The `reject` function has the following parameters:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
code         | 403        | The HTTP response status code.
reason       | 'Rejected' | The HTTP response status reason string.


### WebSocketTransport

Created when calling `accept()` within the `connectionrequest` event of the `WebSocketServer`, it represents an established WebSocket connection with a client.

No public API is exposed.
