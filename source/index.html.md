---
title: protoo

language_tabs:
  - javascript

toc_footers:
  - <a href='https://github.com/versatica/protoo'>Source Code at GitHub</a>

search: true
---

# prot<span class="double-o">oo</span>

_Documentation for v4_

**protoo** is a minimalist and extensible Node.js signaling framework for multi-party Real-Time Communication applications.

It provides both a server side Node.js module and a client side JavaScript library. Its primary purpose is to provide applications with the ability to easily add group chat, presence and multi-party multimedia capabilities.


## Messages

**protoo** defines a signaling protocol based on JSON requests, responses and notifications. It is up to the application to define and extend the signaling protocol and the content of those messages in order to accomplish the desired feature set.


### Request

A **protoo** request is a JSON message.

```javascript
{
  request : true,
  id      : 12345678,
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
id           | A number identifier to match its associated response.
method       | A custom string representing the request method.
data         | An object with custom data.


### Response

A **protoo** response is a JSON message associated to a **protoo** request.

> Success response

```javascript
{
  response : true,
  id       : 12345678,
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
  id          : 12345678,
  ok          : false,
  errorCode   : 123,
  errorReason : 'Something failed'
}
```

Field        | Description
------------ | ------------------------------
response     | Must be `true`.
id           | A number identifier matching its associated request.
ok           | `true` if a success response, `false` otherwise.
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

The top-level module exposes:

* `version` (String)
* `WebSocketServer` (Class)
* `Room` (Class)


### version

The package version.


### WebSocketServer

A `WebSocketServer` listens for WebSocket connections from clients.

```javascript
const options =
{
  maxReceivedFrameSize     : 960000, // 960 KBytes.
  maxReceivedMessageSize   : 960000,
  fragmentOutgoingMessages : true,
  fragmentationThreshold   : 960000
};

const server = new protooServer.WebSocketServer(httpServer, options);
```

Parameter    | Description
------------ | ------------------------------
httpServer   | A Node.js `http.Server` or `https.Server` object.
[options]    | Options for [websocket.WebSocketServer](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketServer.md#server-config-options).

<aside class='notice'>
It's up to the application to set the <code>http.Server</code> or <code>https.Server</code> and call <code>listen()</code> on it.
<br><br>
In this way, the <code>http.Server</code> or <code>https.Server</code> can also be used for pure HTTP purposes by sharing it with <a href='https://expressjs.com/'>Express</a> or any other Node.js HTTP framework.
</aside>


#### `stop()`

Unmounts the WebSocket server so no more WebSocket connections will take place.

<aside class='notice'>
When <code>stop()</code> is called, the underlying Node.js <code>http.Server</code> or <code>https.Server</code> is not closed.
</aside>


#### `on('connectionrequest', fn(info, accept, reject))`

Event fired when a WebSocket client attempts to connect to the WebSocket server.

```javascript
server.on('connectionrequest', (info, accept, reject) =>
{
  // The app inspects the `info` object and decides whether to accept the
  // connection or not.  
  if (something in info)
  {
    const transport = accept();

    // The app chooses a `peerId` and creates a peer within a specific room.
    const peer = room.createPeer('bob', transport);
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
[reason]     | 'Rejected' | The HTTP response status reason string.

or:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
error        |            | An `Error` instance.


### WebSocketTransport

Created when calling `accept()` within the `connectionrequest` event of the `WebSocketServer`. It represents an established WebSocket connection with a client.

No public API is exposed.


### Room

A `Room` represents a multi-party communication context.

```javascript
const room = new protooServer.Room();
```


#### `peers`

Returns an array with all the connected `Peer` instances in the room.

```javascript
for (let peer of room.peers)
{
  console.log('peer id: %s', peer.id);
}
```


#### `closed`

Boolean indicating whether the room is closed.


#### `createPeer(peerId, transport)`

Creates a `Peer` within this room. It returns a `Peer` instance. It throws if wrong parameters are given or if there is already a peer with the same `peerId` in the room.

```javascript
const peer = room.createPeer('alice', transport);
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


#### `close()`

Closes the room and emits `close` event. All the peers within this room will also be closed and their `close` event fired.


#### `on('close', fn())`

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


#### `async request(method, [data])`

Sends a **protoo** request to the peer. It resolves to the `data` object of the **protoo** response (if successful) or rejects if an error response is received (or any other kind of error happens).

```javascript
try
{
  const data = await peer.request('chicken', { foo: 'bar' });

  console.log('got response data:', data);
}
catch (error)
{
  console.error('request failed:', error);
}
```

Parameter    | Description
------------ | ------------------------------
method       | Request method string.
[data]       | Request data object.


#### `async notify(method, [data])`

Sends a **protoo** notification to the peer. It resolves if the notification was correctly sent, and rejects if the notification could not be sent.

```javascript
peer.notify('lalala', { foo: 'bar' });
```

Parameter    | Description
------------ | ------------------------------
method       | Notification method string.
[data]       | Notification data object.


#### `close()`

Closes the peer and its underlying transport, and emits `close` event.


#### `on('request', fn(request, accept, reject))`

Event fired when a **protoo** request is received from the peer.

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
[data]       | `{}`       | The `data` object field of the response.

The `reject` function has the following parameters:

Parameter     | Default    | Description
------------- | ---------- | -----------------------
errorCode     |            | Error numeric code.
[errorReason] |            | Error text description.

or:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
error        |            | An `Error` instance.


#### `on('notification', fn(notification))`

Event fired when a **protoo** notification is received from the peer.

```javascript
peer.on('notification', (notification) =>
{
  // Do something.
});
```

Parameter    | Description
------------ | ------------------------------
notification | A **protoo** notification.


#### `on('close', fn())`

Event fired when the peer is closed by calling `close()` on it, or when the underlying transport is remotely closed, or when the room is closed.


# protoo-client

<a href='https://npmjs.org/package/protoo-client'><img src='https://img.shields.io/npm/v/protoo-client.svg' alt=''></a>

The **protoo** client side JavaScript library. It runs in both browser and Node.js environments.


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

* `version` (String)
* `WebSocketTransport` (Class)
* `Peer` (Class)


### version

The package version.


### WebSocketTransport

A `WebSocketTransport` creates a WebSocket connection.

```javascript

const transport = new protooClient.WebSocketTransport('wss://example.org');
```

Parameter    | Description
------------ | ------------------------------
url          | WebSocket connection URL.
[options]    | Includes the options for [websocket.W3CWebSocket](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/W3CWebSocket.md#constructor) (all but `requestUrl`) plus a `retry` parameter (see below).

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
The <code>options</code> parameters other than `retry` are just valid when using <strong>protoo-client</strong> in Node.js.
</aside>


### Peer

A `Peer` represents a participant in a remote room.

```javascript

const peer = new protooClient.Peer(transport);
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


#### `connected`

Boolean indicating whether the peer is connected.


#### `async request(method, [data])`

Sends a **protoo** request to the peer. It resolves to the `data` object of the **protoo** response (if successful) or rejects if an error response is received (or any other kind of error happens).

```javascript
try
{
  const data = await peer.request('chicken', { foo: 'bar' });

  console.log('got response data:', data);
}
catch (error)
{
  console.error('request failed:', error);
}
```

Parameter    | Description
------------ | ------------------------------
method       | Request method string.
[data]       | Request data object.


#### `async notify(method, [data])`

Sends a **protoo** notification to the peer. It resolves if the notification was correctly sent, and rejects if the notification could not be sent.

```javascript
peer.notify('lalala', { foo: 'bar' });
```

Parameter    | Description
------------ | ------------------------------
method       | Notification method string.
[data]       | Notification data object.


#### `close()`

Closes the peer and its underlying transport, and emits `close` event.


#### `on('open', fn())`

Event fired when the transport is connected.


#### `on('failed', fn(currentAttempt))`

Event fired when the connection with the server fails (due to network errors, not running server, unreachable server address, etc).

The peer will try to connect as many times as defined in its `retry` options. After those retries, the `close` event will fire.

Parameter      | Description
-------------- | ------------------------------
currentAttempt | Reconnection attempt (starts with 1).



#### `on('disconnected', fn())`

Event fired when the established connection is abruptly closed. The peer will start the reconnection procedures as defined in its `retry` options.

The peer will try to reconnect as many times as defined in its `retry` options. After those retries the <code>close</code> event will fire.

<aside class='notice'>
If <code>close()</code> was called in the server-side <code>room</code>, in the server-side <code>peer</code> or in this client-side <code>peer</code>, no reconnection attempts will take place.
</aside>


#### `on('close', fn())`

Event fired when the peer is closed by calling `close()` on it, or when the underlying transport is remotely closed (via `room.close()` or `peer.close()` in server-side), or after all reconnection attempts fail.



#### `on('request', fn(request, accept, reject))`

Event fired when a **protoo** request is received from the room.

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
[data]       | `{}`       | The `data` object field of the response.

The `reject` function has the following parameters:

Parameter     | Default    | Description
------------- | ---------- | -----------------------
errorCode     |            | Error numeric code.
[errorReason] |            | Error text description.

or:

Parameter    | Default    | Description
------------ | ---------- | -----------------------
error        |            | An `Error` instance.


#### `on('notification', fn(notification))`

Event fired when a **protoo** notification is received from the room.

```javascript
peer.on('notification', (notification) =>
{
  // Do something.
});
```

Parameter    | Description
------------ | ------------------------------
notification | A **protoo** notification.
