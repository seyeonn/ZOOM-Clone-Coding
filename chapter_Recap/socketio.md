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



## #2.5 Room Messages

```js
socket.to(roomName).emit("welcome");
```

-> "welcome" event를 roomName에 있는 모든 사람들에게 emit한다.

```js
done();
```

-> done() function은 FrontEnd에 있는 showRoom()을 실행한다.
-> 그 이후에 event를 방금 참가한 방 안에 있는 모든 사람에게 emit해준다.

```js
function addMessage(message) {
    const ul = room.querySelector("ul");
    const li = document.createElement("li");
    li.innerText = message;
    ul.appendChild(li);
}
```
-> 언제든지 메세지를 보낼 수 있는 함수

```js
socket.on("welcome", () => {
    addMessage("someone joined!");
});
```
-> 함수 실행 코드 (FrontEnd)

```js
socket.on("enter_room", (roomName, done) => {
        socket.join(roomName);
        done();
        socket.to(roomName).emit("welcome");
    });
```
-> 함수 실행 코드 (BackEnd)

![image](https://i.imgur.com/kcOLfDg.png)

-> 유저 입장 후 룸 생성 - 또다른 유저 입장 시 'someone joined!' 채팅 실행



## #2.6 Room Notifications

### disconnect vs disconnecting

**disconnect** : 연결이 완전히 끊겼음.
**disconnecting** : 고객이 접속을 중단할 것이지만 아직 방을 완전히 나가지는 않은 것. (창을 닫거나 컴퓨터가 꺼졌을 때 방에 message를 보낼 수 있음.)

```server.js```

```js
socket.on("disconnecting", () => {
        socket.rooms.forEach(room => socket.to(room).emit("bye"));
    })
```

```app.js```

```js
socket.on("bye", () => {
    addMessage("someone left ㅠㅠ");
});
```

- 실행 결과 화면

![image](https://i.imgur.com/YMO9vWf.png)

- 메세지 송수신

```server.js```

```js
socket.on("new_message", (msg, room, done) => {
        socket.to(room).emit("new_message", msg);
        done();
    })
```

```app.js```

```js
function addMessage(message) {
    const ul = room.querySelector("ul");
    const li = document.createElement("li");
    li.innerText = message;
    ul.appendChild(li);
}

function handleMessageSubmit(event) {
    event.preventDefault();
    const input = room.querySelector("input");
    const value = input.value;
    socket.emit("new_message", input.value, roomName, () => {
        addMessage(`You: ${value}`);
    });
    input.value = "";
}

function showRoom() {
    welcome.hidden = true;
    room.hidden = false;
    const h3 = room.querySelector("h3");
    h3.innerText = `Room ${roomName}`;
    const form = room.querySelector("form");
    form.addEventListener("submit", handleMessageSubmit);
}

form.addEventListener("submit", handleRoomSubmit);

socket.on("new_message", addMessage);
```

- 실행 결과 화면

![image](https://i.imgur.com/PzJVmKX.png)



## #2.7 Nicknames

- nickname과 message를 위한 form 생성

```app.js```

```js
function handleNicknameSubmit(event) {
    event.preventDefault();
    const input = room.querySelector("#name input");
    socket.emit("nickname", input.value);
}
```

```js
const nameForm = room.querySelector("#name");

nameForm.addEventListener("submit", handleNicknameSubmit);
```

```server.js```

```js
socket.on("nickname", nickname => (socket["nickname"] = nickname));
```

-> "nickname" event가 발생하면 nickname을 가져와서 socket에 저장한다.

```js
socket.on("new_message", (msg, room, done) => {
        socket.to(room).emit("new_message", `${socket.nickname}: ${msg}`);
        done();
    });
```

-> nickname과 msg 전달

- 실행 결과 화면

![image](https://i.imgur.com/28xEDwF.png)



## #2.8 Room Count part One

### Adapter

- Adapter가 기본적으로 하는 일은 다른 서버들 사이에 실시간 어플리케이션을 동기화 하는 것이다.

- 지금 우리는 서버의 메모리에서 Adapter를 사용하고 있다. 데이터베이스에는 아무것도 저장하고 있지 않다.

- 우리가 서버를 종료하고 다시 시작할 때 모든 room과 message와 socket은 없어진다. (= 재시작할 때 모든 것들이 처음부터 다시 시작된다.)

- 우리는 그래서 백엔드에 데이터베이스를 가지고 싶을 것이다.

- 그리고 앱 안에 많은 클라이언트가 있을 때, 모든 클라이언트에 대해서 connection을 열어둬야 한다.

- 그 연결은 실시간으로 서버에 있어야 한다. 그렇기 때문에 서버는 이 connection을 오픈된 상태를 유지해야 한다.

- 서버 메모리에서 Adapter를 사용하고 있다면 우리가 만든 3개의 서버는 같은 memory pool을 공유하지 않을 것이다.

-> 데이터베이스를 사용해서 Adapter 역할을 잘 활용해보자!


---

- chapter의 목표 : ** socket의 ID를 뜻하는 sids를 가져와서 방들을 보고 이 방들이 어떤 socket을 위해서 만들어졌는지 확인해보자** (private 메세지와 모든 사람들과 채팅하기 위해 만든 방이 어떤건지)

![image](https://i.imgur.com/pxH7FDo.png)

-> Private Rooms 2개와 Public Room 1개 

-> 이 Map에 있는 모든 rooms를 확인하고 room의 id도 확인해보자!

-> 만약 room ID를 socket ID에서 찾을 수 있다면 우리가 Private용 room을 찾은 것이다.

-> 만약 room ID를 socket ID에서 찾을 수 없다면 우리가 Public room을 찾은 것이다.

-> 그걸 하기 위해선 Map 데이터 구조를 봐야 한다. 

![image](https://i.imgur.com/W7gV7vr.png)

-> sids에 대한 map을 만들자. (sids는 우리 BackEnd에 연결된 모든 sockets들의 map이다.)

-> 때때로 socket id는 room id와 같다.

-> 모든 socket은 private room이 있다는 것을 명심하자. 이것은 바로 id가 방제목인 경우이다. 그래서 우리는 private message를 보낼 수 있다.

-> get과 key를 이용해서 key가 socket id인지 아니면 방제목인지 확인할 수 있다.

![image](https://i.imgur.com/aWnxO33.png)

```js
function publicRooms() {
    const {
        sockets: {
            adapter: {sids, rooms},
        },
    } = wsServer;
    const publicRooms = [];
    rooms.forEach((_, key) => {
        if(sids.get(key) === undefined) {
            publicRooms.push(key);
        }
    });
}
```



## #2.9 Room Count part Two

```js
socket.to(roomName).emit("welcome", socket.nickname);
```
-> 메세지를 하나의 socket에만 보낸다.

```js
wsServer.sockets.emit("room_change", publicRooms());
```

-> 메세지를 모든 socket에 보내준다.

- user가 나가면 room도 사라지게 해주자.

```js
socket.on("disconnect", () => {
        wsServer.sockets.emit("room_change", publicRooms());
    });
```

- room_change event가 발생했을 때 rooms배열을 저장해준다.

```js
socket.on("room_change", (rooms) => {
    const roomList = welcome.querySelector("ul");
    rooms.forEach(room => {
        const li = document.createElement("li");
        li.innerText = room;
        roomList.append(li);
    });
})
```

-> 방에 들어가기 전 기다릴 때 열려있는 모든 방의 list를 확인할 수 있다. 

- 실행 결과 화면

![image](https://i.imgur.com/VMASkdV.png)

```js
socket.on("room_change", (rooms) => {
    const roomList = welcome.querySelector("ul");
    if(rooms.length === 0) {
        roomList.innerHTML = "";
        return;
    }
    rooms.forEach(room => {
        const li = document.createElement("li");
        li.innerText = room;
        roomList.append(li);
    });
})
```

-> rooms가 없는 상태로 오면, 즉 내 어플리케이션에 room이 하나도 없을 때 모든 것을 비워주는 코드

-> but, room 또 생성시 전에 저장된 room이 한번 더 저장이 된다. 문제 발생!

-> 이런 일이 없도록 항상 roomList를 비워주도록 하자. 늘 새로운 list가 되도록!



## #2.10 User Count

- set의 사이즈를 구해보자.

![image](https://i.imgur.com/XmOaKej.png)

- user는 입장하고 퇴장할 때 인원수가 수정된다.

- 수를 세주는 function을 만들어보자.

```js
function countRoom(roomName) {
    return wsServer.sockets.adapter.rooms.get(roomName)?.size;
}
```

-> 중간에 '?'는 가끔 roomName을 찾을 수 있지만 아닐 수도 있기 때문에 넣어줬다. 
-> 우리 방이 얼마나 큰지 계산한다. 몇명이 들어와있는지. 
-> welcome event를 보낼 때  countRoom의 결과값도 같이 보낸다는 의미이다.

```server.js```

```js
socket.to(roomName).emit("welcome", socket.nickname, countRoom(roomName));
```

```js
socket.to(room).emit("bye", socket.nickname, countRoom(room)-1));
```
-> 아직 방을 떠나지 않았기 때문에 현재 서버도 포함되어서 계산이 된다. 그렇기 때문에 현재 room을 빼고 countRoom(room)-1 로 계산해주어야 한다.

-> 즉, welcome과 bye를 할 때 newCount를 받는다.

```app.js```

```js
socket.on("welcome", (user, newCount) => {
    const h3 = room.querySelector("h3");
    h3.innerText = `Room ${roomName} (${newCount})`;
    addMessage(`${user} arrived!`);
});

socket.on("bye", (left, newCount) => {
    const h3 = room.querySelector("h3");
    h3.innerText = `Room ${roomName} (${newCount})`;
    addMessage(`${left} left ㅠㅠ`);
});
```

- 실행 결과 화면

![image](https://i.imgur.com/l9cZyxL.png)



