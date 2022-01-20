# #2 SOCKETIO

## #2.0 SocketIO vs WebSockets

### SocketIO

- 역할 : 실시간, 양방향, event 기반의 통신이 가능하다.(webSocket과 동일.)

- 프레임워크로 webSocket보다 탄력성이 뛰어나다.

- webSocket보다 더 큰 개념. (webSocket은 socketIO의 한 일부분이다.)

- SocketIO는 webSocket의 부가 기능이 아니다.

- socketIO는 webSocket을 사용한다. 하지만 webSocket을 지원하지 않는다면 HTTP long-polling 같은 다른 것을 사용한다.

- webSocket에 문제가 있거나 브라우저가 webSocket을 지원하지 않는다거나 등 어떤 경우에도 SocketIO는 실시간 기능을 제공해준다.

- SocketIO는 프론트와 백엔드 간 실시간 통신을 가능하게 해주는 프레임워크 또는 라이브러리이다.

- 자동 재연결 지원, 연결 끊김 확인, 바이너리 지원



## #2.1 Installing SocketIO

- socketIO 설치

```js
npm i socket.io
```

```js
import SocketIO from "socket.io";

const httpServer = http.createServer(app);
const wsServer = SocketIO(httpServer);
```

- 브라우저가 주는 websocket은 socketIO와 호환이 되지 않기 때문에(socketIO가 더 많은 기능을 제공하기 때문) socketIO를 브라우저에 설치해야 한다.

```js
script(src="/socket.io/socket.io.js")
```

- connection을 받을 준비를 한다.

```server.js```

```js
wsServer.on("connection", (socket) => {
    console.log(socket);
});
```

```app.js```

```js
const socket = io();
```

-> io function은 알아서 socket.io를 실행하고 있는 서버를 찾는다.
-> socketIO를 frontEnd와 연결할 수 있다.

- 결과 실행 화면

![image](https://i.imgur.com/jEAd7Ta.png)

-> 현재 연결된 모든 socket을 자동적으로 추적하고 있다. (webSocket에서는 sockets.push(socket)을 했어야 했음.)



