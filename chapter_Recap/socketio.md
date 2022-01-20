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



## #2.2 SocketIO is Amazing

- room의 개념 :socketIO를 이용해서 방에 참가하고 떠나는 것을 해보자.

- 방의 유무와 상관없이 그냥 방에 들어간다.

- webSocket과 socketIO의 차이

```webSocket```

```js
socket.send()
```

```SocketIO```

```js
socket.emit()
```
-> 그리고 메세지를 보내줄 필요 없다.
-> emit은 argument를 보낼 수 있고, argument는 object가 될 수 있다.

### webSocket

1. object를 string으로 변환시키고 그 후에 string을 전송할 수 있었다.

### socketIO ✔

1. socketIO는 저걸 다 알아서 해준다.

2. 특정한 event를 emit해줄 수 있다. 어떤 이름이든 상관없음.

3. object를 전송할 수 있다.

4. message event가 아니어도 되고 어떤 event든 다 가능하다.

5. JavaScript Object를 전송할 수 있다.

6. 서버로부터 실행되는 function인 callback 기능 제공


- room 연결

```app.js```

```js
const welcome = document.getElementById("welcome");
const form = welcome.querySelector("form");

function handleRoomSubmit(event) {
    event.preventDefault();
    const input = form.querySelector("input");
    socket.emit("enter_room", {payload: input.value});
    input.value = "";
}

form.addEventListener("submit", handleRoomSubmit);
```

```server.js```

```js
wsServer.on("connection", (socket) => {
    socket.on("enter_room", (msg) => console.log(msg));
});
```

- 실행 결과 화면

![image](https://i.imgur.com/vVQzd0c.png)

-> enter room 버튼을 클릭하면 socket.emit()이 실행된다.

![imahe](https://i.imgur.com/apDu6Aj.png)

-> socketIO를 이용해서 메세지를 전송하고 있다.

- socket.emit에서 첫번째 argument에는 event 이름이 들어가고, 두번째 argument에는 보내고 싶은 payload가 들어간다. 세번째 argument에는 서버에서 호출하는 function이 들어간다.

```js
socket.emit("enter_room", {payload: input.value}, () => {
        console.log("server is done!");
    });
```

- **emit과 on은 같은 이름, 같은 string이어야 한다.**

- 서버는 backEnd에서 function을 호출하지만 function은 frontEnd에서 실행될 수 있다.



