---
title: protoo

language_tabs:
  - javascript

toc_footers:
  - <a href='https://github.com/versatica/protoo'>Source Code at GitHub</a>

search: true
---

# prot<span class="double-o">oo</span>

**protoo** is a minimalist and extensible Node.js signaling framework for multi-party Real-Time Communication applications.

It provides both a server side Node.js module and a client side JavaScript library. Its primary purpose is to provide applications with the ability to easily add group chat, presence and multi-party multimedia capabilities.


## Messages

**protoo** defines a signaling protocol based on JSON requests, responses and notifications. It is up to the application to define and extend the signaling protocol and the content of those messages in order to accomplish the desired feature set.


### Request

A **protoo** request is a JSON message.

```javascript
{
  request : true,
  id      : 'kj3487asjh',
  method  : 'chatmessage',
  data    :
  {
    type  : 'text',
    value : 'Hi there!'
  }
}
```

Field        | Description
------------ | ------------------------------
request      | Must be `true`.
id           | A string identifier to match its associated response.
method       | A custom string representing the request method.
data         | An object with custom data.


### Response

A **protoo** response is a JSON message associated to a **protoo** request.

> Success response

```javascript
{
  response : true,
  id       : 'kj3487asjh',
  ok       : true,
  data     :
  {
    foo : 'lalala'
  }
}
```

> Error response

```javascript
{
  response    : true,
  id          : 'kj3487asjh',
  errorCode   : 123,
  errorReason : 'Something failed'
}
```

Field        | Description
------------ | ------------------------------
response     | Must be `true`.
id           | A string identifier matching its associated request.
ok           | `true` if a success response.
data         | An object with custom data (just for success responses).
errorCode    | Numeric error code up to the application (just for error responses).
errorReason  | Descriptive error text up to the application (just for error responses).


### Notification

A **protoo** notification is a JSON message (so there is no associated response).

```javascript
{
  notification : true,
  method       : 'chatmessage',
  data         :
  {
    foo : 'bar'
  }
}
```

Field        | Description
------------ | ------------------------------
notification | Must be `true`.
method       | A custom string representing the notification method.
data         | An object with custom data.


# protoo-server

<a href='https://npmjs.org/package/protoo-server'><img src='https://img.shields.io/npm/v/protoo-server.svg' alt=''></a>

The **protoo** server side Node.js module.


## Installation

Install the [protoo-server](https://www.npmjs.com/package/protoo-server) NPM package into your Node.js server side application.

```bash
$ npm install --save protoo-server
```


## API

```javascript
const protooServer = require('protoo-server');
```

The top-level module exposes two JavaScript classes:

* `WebSocketServer`
* `Room`


### WebSocketServer

A `WebSocketServer` listens for WebSocket connections from clients.

```javascript
let options =
{
  maxReceivedFrameSize     : 960000, // 960 KBytes.
  maxReceivedMessageSize   : 960000,
  fragmentOutgoingMessages : true,
  fragmentationThreshold   : 960000
};

let server = new protooServer.WebSocketServer.Room(httpServer, options);
```

Parameter    | Description
------------ | ------------------------------
httpServer   | A Node.js `http.Server` or `https.Server` object.
options      | Options for [websocket.WebSocketServer](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketServer.md#server-config-options).

<aside class='notice'>
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

Event fired when a WebSocket client attempts to connect to the WebSocket server. The `listener` function is called with the following parameters:

```javascript
server.on('connectionrequest', (info, accept, reject) =>
{
  // The app inspects the `info` object and decides whether to accept the
  // connection or not.  
  if (something in info)
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

Parameter    | Description
------------ | ------------------------------
info         | An object with information about the connection attempt.
accept       | Function to be called if the connection must be accepted.
reject       | Function to be called if the connection must be rejected.

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

or:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
error        |            | An `Error` instance.


### WebSocketTransport

Created when calling `accept()` within the `connectionrequest` event of the `WebSocketServer`, it represents an established WebSocket connection with a client.

No public API is exposed.


### Room

A `Room` represents a multi-party communication context.

```javascript
let room = new protooServer.Room();
```


#### `peers`

Returns an array with all the `Peer` instances in the room.

```javascript
for (let peer of room.peers)
{
  console.log('peer id: %s', peer.id);
}
```


#### `closed`

Boolean indicating whether the room is closed.


#### `createPeer(peerId, transport)`

Creates a peer within this room. It returns the new `Peer` instance. It may throw if wrong parameters are given or if there is already a peer with the same `peerId` in the room.

```javascript
let peer = room.createPeer('alice', transport);
```

Parameter    | Description
------------ | ------------------------------
peerId       | Unique string identifier for the peer.
transport    | A `WebSocketTransport` instance.


#### `hasPeer(peerId)`

Returns `true` if there is a peer in the room with the given `peerId`.

Parameter    | Description
------------ | ------------------------------
peerId       | Peer string identifier.


#### `getPeer(peerId)`

Returns the `Peer` instance with the given `peerId`, or `undefined` if not present in the room.

Parameter    | Description
------------ | ------------------------------
peerId       | Peer string identifier.


#### `spread(method, data, excluded)`

Send a notification to all the peers in the room.

```javascript
room.spread('notification', data);
```

Parameter    | Description
------------ | ------------------------------
method       | Notification method string.
data         | Notification data object.
excluded     | Optional array of `Peer` instances or `peerId` string values who won't receive the request.


#### `close()`

Closes the room and emits `close` event. All the peers within this room will also be closed.


#### `on('close', listener)`

Event fired when the room is closed.


### Peer

A `Peer` represents a remote client connected to a `Room`.


#### `id`

Read-only string identifier of the peer.


#### `data`

Writable custom object up to the application.

```javascript
peer.data.foo = 'bar';

console.log(peer.data.foo);
```


#### `closed`

Boolean indicating whether the peer is closed.


#### `send(method, data)`

Send a request to the peer. It returns a Promise resolving to the `data` object field of the response (if successful).

```javascript
peer.send('chicken', { foo: 'bar' })
  .then((data) =>
  {
    console.log('success response received');
  })
  .catch((error) =>
  {
    console.error('error response');
  });
```

Parameter    | Description
------------ | ------------------------------
method       | Request method string.
data         | Request data object.


#### `notify(method, data)`

Send a notification to the peer. It returns a Promise that resolves if the notification was correctly sent.

```javascript
peer.notify('lalala', { foo: 'bar' })
  .catch((error) =>
  {
    console.error('could not send notification');
  });
```

Parameter    | Description
------------ | ------------------------------
method       | Notification method string.
data         | Notification data object.


#### `close()`

Closes the peer and its underlying transport, and emits `close` event.


#### `on('request', listener)`

Event fired when a request is received from the peer. The `listener` function is called with the following parameters:

```javascript
peer.on('request', (request, accept, reject) =>
{
  if (something in request)
    accept({ foo: 'bar' });
  else
    reject(400, 'Not Here');
});
```

Parameter    | Description
------------ | ------------------------------
request      | A **protoo** request.
accept       | Function to be called if the request must be accepted.
reject       | Function to be called if the request must be rejected.

The `accept` function has the following parameters:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
data         | `{}`       | The `data` object field of the response.

The `reject` function has the following parameters:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
errorCode    |            | Error numeric code.
errorReason  |            | Error text description.

or:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
error        |            | An `Error` instance.


#### `on('notification', listener)`

Event fired when a notification is received from the peer. The `listener` function is called with the following parameters:

```javascript
peer.on('notification', (notification) =>
{
  // Do something.
});
```

Parameter    | Description
------------ | ------------------------------
notification | A **protoo** notification.


#### `on('close', listener)`

Event fired when the peer is closed by calling `close()` on it, or when the underlying transport is remotely closed.


# protoo-client

<a href='https://npmjs.org/package/protoo-client'><img src='https://img.shields.io/npm/v/protoo-client.svg' alt=''></a>

The **protoo** client side JavaScript library. It runs in both the browser (by using <a href='http://browserify.org/'>Browserify</a>) and Node.js environments.

<aside class='notice'>
The library is written in JavaScript ES6 so, when running in the browser, <a href='https://babeljs.io/'>Babel</a> may be required depending on the browser.
</aside>

## Installation

Install the [protoo-client](https://www.npmjs.com/package/protoo-client) NPM package into your client side application.

```bash
$ npm install --save protoo-client
```


## API

```javascript
const protooClient = require('protoo-client');
```

The top-level module exposes two JavaScript classes:

* `WebSocketTransport`
* `Peer`


### WebSocketTransport

A `WebSocketTransport` creates a WebSocket connection.

```javascript

let transport = new protooClient.WebSocketTransport('wss://example.org');
```

Parameter    | Description
------------ | ------------------------------
url          | WebSocket connection URL.
options      | Includes the options for [websocket.W3CWebSocket](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/W3CWebSocket.md#constructor) (all but `requestUrl`) plus a `retry` parameter (see below).

The `retry` parameters matches the `options` object given to [`retry.operation()`](https://www.npmjs.com/package/retry#retryoperationoptions) and controls the connection and reconnection attempts.

> If `options.retry` is not given, it defaults to the following values

```javascript
{
  retries    : 10,
  factor     : 2,
  minTimeout : 1 * 1000,
  maxTimeout : 8 * 1000
};
```

<aside class='notice'>
The <code>options</code> parameter is just valid when using <strong>protoo-client</strong> in Node.js.
</aside>


### Peer

A `Peer` represents a participant in a remote room.

```javascript

let peer = new protooClient.Peer(transport);
```

Parameter    | Description
------------ | ------------------------------
transport    | A `WebSocketTransport` instance.


#### `data`

Writable custom object up to the application.

```javascript
peer.data.bar = 1234;

console.log(peer.data.bar);
```


#### `closed`

Boolean indicating whether the peer is closed.


#### `send(method, data)`

Send a request to the room. It returns a Promise resolving to the `data` object field of the response (if successful).

```javascript
peer.send('hello', { lalala: 'foo' })
  .then((data) =>
  {
    console.log('success response received');
  })
  .catch((error) =>
  {
    console.error('error response');
  });
```

Parameter    | Description
------------ | ------------------------------
method       | Request method string.
data         | Request data object.


#### `notify(method, data)`

Send a notification to the room. It returns a Promise that resolves if the notification was correctly sent.

```javascript
peer.notify('lalala', { foo: 'bar' })
  .catch((error) =>
  {
    console.error('could not send notification');
  });
```

Parameter    | Description
------------ | ------------------------------
method       | Notification method string.
data         | Notification data object.


#### `close()`

Closes the peer and its underlying transport, and emits `close` event.


#### `on('open', listener)`

Event fired when the peer is connected to the room.


#### `on('request', listener)`

Event fired when a request is received from the room. The `listener` function is called with the following parameters:

```javascript
peer.on('request', (request, accept, reject) =>
{
  if (something in request)
    accept({ foo: 'bar' });
  else
    reject(400, 'Not Here');
});
```

Parameter    | Description
------------ | ------------------------------
request      | A **protoo** request.
accept       | Function to be called if the request must be accepted.
reject       | Function to be called if the request must be rejected.

The `accept` function has the following parameters:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
data         | `{}`       | The `data` object field of the response.

The `reject` function has the following parameters:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
errorCode    |            | Error numeric code.
errorReason  |            | Error text description.

or:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
error        |            | An `Error` instance.


#### `on('notification', listener)`

Event fired when a notification is received from the room. The `listener` function is called with the following parameters:

```javascript
peer.on('notification', (notification) =>
{
  // Do something.
});
```

Parameter    | Description
------------ | ------------------------------
notification | A **protoo** notification.


#### `on('close', listener)`

Event fired when the peer is closed by calling `close()` on it, or when the underlying transport is remotely closed.
