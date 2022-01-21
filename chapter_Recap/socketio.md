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



## #2.3 Recap

1. socketIO를 이용하면, 모든 것이 message일 필요가 없다. (여러 type의 message가 생기면 message function이 엄청 커지기 때문에 힘들어짐.) 

2. client는 너가 원하는 **어떠한 event든 모두** ```emit```해줄 수 있다.

3. 전송할 때 정말 우리가 **원하는 아무거나 전송**이 가능하다. (그 전에는 text만 전송할 수 있었음.)

4. 한 가지만 보내야한다는 제약이 없다. 우리가 **원하는 만큼 전송**할 수 있다.

5. 연결 이상이 감지되면 재연결을 자동으로 시도해준다. 연결이 끊어졌다가 다시 연결되면 에러가 자동으로 사라진다.

6. ```socket.emit```과 ```socket.on```에는 **같은 이름**을 사용해야 한다.

7. socket에서 메세지를 받고 싶을 때, 처리 비용이 크고 시간이 오래걸리는 작업이어도 frontEnd에 작업을 완료했다고 알리고 싶을 때 등 function을 보내고 싶으면 해당 function을 마지막 argument에 넣으면 된다. backEnd가 그 코드를 실행시키는 것이 아니고 (보안상 문제가 생김) backEnd에서 function을 실행시켰을 때 frontend에서 function을 실행시킬 것이다.

8. frontEnd에서 실행 된 코드는 backEnd가 실행을 시킨 것이다.



## #2.4 Rooms

- socketIO는 기본적으로 room을 제공한다.

- socket에는 id가 있다. 

- socket.onAny(callback)은 어느 event에서든지 console.log를 할 수 있다.

```js
socket.onAny((event) => {
        console.log(`Socket Event : ${event}`);
    });
```

```js
 console.log(socket.rooms);
        socket.join(roomName);
        console.log(socket.rooms);
```

- 1212 입력하고 버튼 누른 후 결과

![image](https://i.imgur.com/FKEeo0A.png)

- user의 id는 user가 있는 방의 id와 같다. 왜냐하면 socketIO에서 모든 socket은 기본적으로 User와 서버 사이에 private room이 있기 때문이다.

- 즉, 그냥 방에 들어가기 위해서 socket.join을 하면 된다.

- socket이 어떤 방에 있는지 알기 위해서는 socket.rooms를 하면 된다.

- socket에는 id가 있어서 id로 구별이 가능하다.

- 방 접속을 구현해보자.

```server.js```

```js
wsServer.on("connection", (socket) => {
    socket.onAny((event) => {
        console.log(`Socket Event: ${event}`);
    });
    socket.on("enter_room", (roomName, done) => {
        socket.join(roomName);
        done();
    });
});
```

```app.js```

```js
const room = document.getElementById("room");

room.hidden = true;

let roomName;

function showRoom() {
    welcome.hidden = true;
    room.hidden = false;
    const h3 = room.querySelector("h3");
    h3.innerText = `Room ${roomName}`;
}
function handleRoomSubmit(event) {
    event.preventDefault();
    const input = form.querySelector("input");
    socket.emit("enter_room", input.value, showRoom);
    roomName = input.value;
    input.value = "";
}

form.addEventListener("submit", handleRoomSubmit);
```

```home.pug```

```js
main
            div#welcome
                form
                    input(placeholder="room name", required, type="text")
                    button Enter Room
            div#room
                h3  
                ul
                form
                    input(placeholder="message", required, type="text")
                    button Send
        script(src="/socket.io/socket.io.js")
```













