---
title: "WebSocket in Angular"
date: 2022-07-14 13:28:00 +0900
comments: true
categories: angular
tags: [websocket, webworker]
---

Let's explore how to use WebSockets with Angular. Due to the extensive nature of the topic, this guide will focus on utilizing standard WebSockets provided by JavaScript. A subsequent discussion will delve into leveraging RxJS for WebSocket implementation.<br/>

A crucial consideration with WebSockets is the protocol requirement: **HTTPS sites must use `wss`, while HTTP sites should use `ws`**.

## Backend Setup

For testing purposes, we'll construct a simple backend using Node.js.

### Node.js Project Initialization

First, initialize a new Node.js project by executing the following command:

```bash
npm init 
```

To enable WebSocket functionality, install the `ws` package:

```bash
npm i ws
```

Create a JavaScript file (e.g., `index.js`) to house the server code. The setup is now complete.

### Initial Code Structure

The primary task is to import `ws` and configure the WebSocket server. This section presents a fundamental code snippet, omitting extraneous details. While the port can be chosen arbitrarily, it's advisable to avoid reserved ports (e.g., 8080, 443).

```js
const WebSocket = require('ws')

const wss = new WebSocket.Server({ port: 8001 });
```

This code establishes a WebSocket connection through port 8001.

### Handling Connections

The `wss` variable, representing the WebSocket server, is used to listen for connection events. All WebSocket-related logic should reside within this connection event handler.

```js
wss.on('connection', ws => {

})
```

When a client initiates a connection (handshake), a connection event is triggered. You can verify this by logging a message to the console within the event handler.

### Receiving Client Messages

To receive messages sent by the client, use the `message` event handler.

```js
wss.on('connection', ws => {
  ws.on('message', message => {

  })
})
```

### Sending Messages: `send`

To transmit messages to a client, use the `send` method.

```js
ws.send(
  JSON.stringify({data: 'message from server'})
  
  );
```
The `message` parameter accepts data of any type. However, due to browser inconsistencies, particularly with older browsers that may only accept strings, it's common practice to serialize data (e.g., using `JSON.stringify`) and parse it on the client-side.<br/>

The `send` method transmits messages to a specific client, enabling individualized communication with each connected client.

### Handling Connection Closure

A `close` event is emitted when a client or server connection is terminated. Strictly speaking, the event is triggered just before the connection closes, allowing for a final message to be sent to the client.

```js
ws.on('close', (code, reason) => {

})
```

### Error Handling

An `error` event is triggered if issues arise during the connection.

```js
ws.on('error', (err) => {

})
```

### Broadcasting Messages

It's possible to send messages to all connected clients simultaneously.

Unlike the `send` method, there isn't a dedicated "broadcast" command. Instead, implement broadcasting by iterating over all connected clients and sending the message to each one individually. The `wss` variable stores information about all connected clients. The following code demonstrates how to retrieve this information and send a message to each client:

```js
wss.on('connection', ws => {
  wss.clients.forEach(client => {
    client.send();
  })
})
```

## Frontend Implementation

This section focuses on the service responsible for establishing the WebSocket connection.

### Connection Establishment

Establishing a connection on the frontend is relatively straightforward. Similar to the backend, event handling code should be added after the connection is opened. The `WebSocket` object provides fundamental state values:

- 0: CONNECTING
- 1: OPEN
- 2: CLOSING
- 3: CLOSED

<br/>

Communication with the server can only occur after the connection state transitions to `1` (OPEN). Ensure this state is verified before sending or receiving messages.

```tsx
@Injectable({
  providedIn: 'root',
  standalone: true,
})
export class NetworkService {
  ws = new WebSocket('ws://localhost:8001');

  connect() {
    if(this.ws.readyState === WebSocket.OPEN) {
      
    }
  }
  
}
```

### ending Messages to the Server

Use the `send` method, mirroring the backend implementation. While the `send` method accepts any data type, it's advisable to serialize data (e.g., using `JSON.stringify`) for consistency, and handle parsing on the backend.

```tsx
  connect() {
    if(this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(
        JSON.stringify({data: 'message from client'})
      )   
    }
  }
```


### Receiving Server Messages

To receive messages from the backend, listen for the `onmessage` event. The `onmessage` event handler receives a `MessageEvent` object as a parameter, which contains the data sent by the server.

```tsx
  ...
  this.ws.onmessage = (e: MessageEvent) => {
    const response: NetworkResponse = JSON.parse(e.data);
  }
```

### Handling Connection Closure

If the server sends a message upon connection closure, the `onclose` event handler can be used to receive it. Note that the `onclose` event handler receives a `CloseEvent` parameter, which differs from the `MessageEvent`.

```tsx
  ...
  this.ws.onclose = (e: CloseEvent) => {
    ...
  }
```

If no message is sent upon closure, the `ws.readyState` property can be compared to `WebSocket.CLOSING` or `WebSocket.CLOSED` to determine the connection state.

```tsx
if(this.ws.readyState === WebSocket.CLOSING) { // or WebSocket.CLOSE
  ...
}
```

## References

- [Websocket Principles Learned While Implementing a Chat Service (feat. node.js)](https://hudi.blog/websocket-with-nodejs/)
- [Simple Websocket Example with Nodejs](https://www.js-tutorials.com/nodejs-tutorial/simple-websocket-example-with-nodejs/)