---
title: "WebSocket in Angular"
date: 2022-07-14 13:28:00 +0900
comments: true
categories: angular
tags: [websocket, webworker]
---

websocket을 angular 사용하는 방법에 대해 알아봅시다.<br/>
모든 내용을 다루기에는 양이 많으므로 이번에는 js에서 제공하는 일반 websocket을 활용하는 방법만 정리하고, <br/>
다음 시간에 rxjs의 websocket을 활용하는 방법에 대해 정리하도록 하겠습니다.<br/><br/>

websocket은 주의할 점이 만일 해당 사이트가 https인 경우에는 wss를, http인 경우에는 ws를 사용해야 합니다.<br/>


## Backend 설정
테스트를 위한 백엔드가 필요합니다. 간단하게 작성할 예정이므로 nodejs를 활용하겠습니다.<br/>


### nodejs 프로젝트 설치

먼저 node 프로젝트 시작을 위해 다음의 명령어를 실행합니다.<br/>

```
npm init 
```

이제 websocket을 사용하려면 ws를 install 해야 하므로 ws도 install 해 줍니다.<br/>

```
npm i ws
```

server 코드를 작성하기 위한 js 파일을 하나 생성하면 설치는 모두 완료됩니다.<br/>
여기에서는 index.js로 작성하였습니다.<br/>

### initial code

가장 먼저 작성해야할 코드는 ws를 가져오는 것과, WebSocket을 설정하는 일입니다.<br/>
다른 복잡한 설명은 제거하고, 여기에서는 가장 기본적인 코드만 작성합니다.<br/>
아래의 코드에서 포트는 임의로 작성해도 되나 특정 포트(8080,443 등)는 피하는 것이 좋습니다. <br/>

```js
const WebSocket = require('ws')

const wss = new WebSocket.Server({ port: 8001 });
```

이로써 8001번 포트를 통해 websocket이 연결되었습니다.<br/>


### connection

이제 websocket 정보를 가진 변수인 wss를 통해 connection 이벤트를 가져와야 합니다.<br/>
당연한 말이겠지만 websocket을 처리하는 코드는 모두 connection 안에 들어 있어야 합니다.<br/>

```js
wss.on('connection', ws => {

})
```

client에서 최초로 연결(handshake)하게 되면 connection 이벤트가 발생합니다.<br/>
이벤트 안에 콘솔을 찍어보면 더 확실하게 확인할 수 있습니다.<br/>


### receive a client message

client가 메시지를 전송하면 해당 메시지를 받을 때 사용합니다.<br/>

```js
wss.on('connection', ws => {
  ws.on('message', message => {

  })
})
```

### send a message : send

client에게 message를 전송합니다. <br/>

```js
ws.send(
  JSON.stringify({data: 'message from server'})
  
  );
```
message는 any형이므로 어떠한 메시지를 전송해도 됩니다. <br/>
단, 브라우저마다 기준이 다르고, 오래된 브라우저의 경우 string으로만 받을 수 있으므로,<br/>
일반적으로는 string 특별히 json을 stringify하여 전달하고 client가 이를 parse하는 방법을 활용합니다.<br/><br/>


send 명령의 특징은 1명의 특정 client에게만 메시지를 전송하는 것입니다.<br/>
다시 말해 연결된 각각의 client에게 개별적으로 전달하는 메시지 입니다.<br/>

### close

클라이언트 또는 서버의 연결이 끊어지면 즉시 close 이벤트가 발생합니다.<br/>
엄밀히는 닫히기 직전에 이벤트가 발생하므로 이 때에도 client에 메시지를 전달할 수 있습니다.<br/>

```js
ws.on('close', (code, reason) => {

})
```

### error
연결에 문제가 있는 경우 또한 이를 감지하여 error 이벤트가 발생합니다.<br/>

```js
ws.on('error', (err) => {

})
```

### broadcast
연결된 모든 client에게 메시지를 전달할 수도 있습니다.<br/>
send와 다른 점은 동시에 모든 client에게 전달하는 것입니다.<br/><br/>

다만 broadcast를 위한 별도의 명령어는 없고, 모든 client에게 send 하는 방식으로 구현해야 합니다.<br/>
모든 client의 정보는 wss 변수에 담겨 있으므로 이를 가져와 각각의 client에게 send 하는 코드를 작성해보겠습니다.<br/>

```js
wss.on('connection', ws => {
  wss.clients.forEach(client => {
    client.send();
  })
})
```

## frontend 
이번 시간에는 component 코드는 생략하고, 직접 websocket을 연결하는 service만 다루도록 하겠습니다.<br/><br/>


### connection
frontend에서는 연결에 크게 추가할 내용은 아래와 같이 간단합니다.<br/>
backend에서는 connection 이후에 코드가 들어가야 하듯 frontend에서도 연결이 open 된 이후에 이벤트 코드가 추가되어야 합니다.<br/>
WebSocket에서는 기본적인 값을 제공하는데 다음과 같습니다.<br/>

- 0: CONNECTING
- 1: OPEN
- 2: CLOSING
- 3: CLOSED
<br/>
서버와의 통신은 1번 OPEN 이후에만 발생하므로 이를 확인하여 작성해야 합니다.<br/>

```tsx
@Injectable({
  providedIn: 'root'
})
export class NetworkService {
  ws = new WebSocket('ws://localhost:8001');

  connect() {
    if(this.ws.readyState === WebSocket.OPEN) {
      
    }
  }
  
}
```

### send message to server

backend의 명령과 같이 send 명령을 사용합니다.<br/>
전송할 값은 any이나 앞서 말한 것과 같이 stringify한 데이터를 전송하고 이를 backend에서 parse하는 방법을 활용합니다.<br/>

```tsx
  connect() {
    if(this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(
        JSON.stringify({data: 'message from client'})
      )   
    }
  }
```


### receive a server message

backend에서 전달된 메시지를 받는 방법은 onmessage 이벤트를 받는 것입니다.<br/>
onmessage는 MessageEvent를 파라미터로 받는데 이 안에 서버에서 전달된 메시지가 들어 있습니다.<br/>

```tsx
  ...
  this.ws.onmessage = (e: MessageEvent) => {
    const response: NetworkResponse = JSON.parse(e.data);
  }
```

### close 
websocket 연결이 끊어졌을 때 만일 server가 어떠한 메시지를 보내기로 했다면,<br/>
onclose 이벤트를 통해서 이를 받을 수 있습니다.<br/>
이 때는 CloseEvent라는 파라미터를 받는데 이는 MessageEvent와 차이가 있으므로 유의하여야 합니다.<br/>

```tsx
  ...
  this.ws.onclose = (e: CloseEvent) => {
    ...
  }
```

만일 별도의 메시지가 없다면 ws.readyState의 값을 비교하여 처리할 수도 있습니다.<br/>

```tsx
if(this.ws.readyState === WebSocket.CLOSING) { // or WebSocket.CLOSE
  ...
}
```

## 참고사이트
- [채팅서비스를 구현하며 배워보는 Websocket 원리 (feat. node.js)](https://hudi.blog/websocket-with-nodejs/)
- [Simple Websocket Example with Nodejs](https://www.js-tutorials.com/nodejs-tutorial/simple-websocket-example-with-nodejs/)