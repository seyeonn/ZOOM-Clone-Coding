# #1 CHAT WITH WEBSOCKETS

## #1.1 HTTP vs WebSockets

![image](https://blog.scaleway.com/content/images/2021/02/websockets-bigger-4.png)


### HTTP

HTTP는 **stateless**하기 때문에 BackEnd가 유저를 기억하지 못한다.

이 말은 즉슨 **유저와 BackEnd 사이에 아무런 연결이 없다**는 뜻이다.

**서버는 유저가 누구인지 까먹는다.** 그렇기 때문에 response를 보낼 때 서버가 일치하는 정보의 cookie를 이용하여 데이터를 전달한다. 이 속성을 **stateless**라고 한다.

response 후에 서버는 잊어버리고 서버는 오직 request를 받을 때만 response를 해준다. 이것이 바로 단방향 통신이다.

그렇기 때문에 HTTP 프로토콜에서는 **request와 response의 반복 과정**이 일어나고 real-time으로는 일어나지 않는다.

 

### WebSocket

브라우저가 서버로 **webSocket request**를 보내면, 서버가 받거나 거절을 하거나 선택하는데 **수락할 시에 연결이 성립**된다. 그럼 이제 **브라우저와 서버는 커뮤니케이션**하게 된다. 

**즉, 연결되어있기 때문에 서버는 유저가 누구인지 기억할 수 있다.** 서버가 유저에게 메세지를 보낼수도 있다.

서버는 request를 기다리지 않아도 되고 **언제 어느때나 메세지를 보낼 수 있다.** 유저도 마찬가지.

request, response 과정이 필요하지 않고, 그냥 **실시간으로 반응**이 되는 것이다. 이것이 바로 양방형 연결이다.



## #1.2 WebSockets in NodeJS

- ```ws```는 **webSocket의 core**이다. 
- webSocket의 핵심적인 패키지를 제공한다.
- express방식은 ws를 지원하지 않는다. function을 추가해야 한다.


- 서버에 접근 가능 (이 서버에서 webSocket 만들 수 있다)

![image](https://i.imgur.com/zs4LClo.png)

- WebSocket 서버를 만들어보자.

```js
const wss = new WebSocket.Server({ server });
```
-> http 서버, webSocket 서버 둘다 돌릴 수 있다. (필수는 아님)
-> 2개가 같은 port에 있길 원해서 하는 것.

- http 서버 위에 WebSocket 서버를 만들 수 있도록 한다. (= 내 http 서버에 access 하려는 것)

- http 서버가 있으면 그 위에서 ws서버를 만들 수 있고 2개의 protocol은 다 같은 prot를 공유할 수 있다.

- 우리의 서버는 http protocol과 ws connection을 지원한다.

![image](https://i.imgur.com/bMBiH29.png)



## #1.3 WebSocket Events

- ```on``` - ```connection``` : 누군가가 우리와 연결했을 때

```js
wss.on("connection", handleConnection)
```

-> 이 method의 설명을 보면, callback으로 socket을 받는다고 한다. (socket은 연결된 어떤 사람, 연결된 브라우저와의 contact(연락)라인)

-> socket을 이용하면 메세지 주고 받기가 가능하다. 이것을 어딘가에 저장해야 한다.

-> on mehtod에서는 event가 실행되는 것을 기다리고 그 event는 connection이 된다. 그리고 또 function을 받는데 connection이 이루어지면 작동한다.

-> 또한 on method는 backend에 연결된 사람의 정보를 제공해준다. 그것이 바로 socket으로 전달되는 것이다.

-> **socket은 서버와 브라우저 사이의 연결이다.** 

-> 이 방식은 event를 listen하고 어딘가에 handler를 만들었을때만 사용한다. (아무 연결이 없기 때문에 아무 일도 일어나지 않는다.)

- frontEnd에서 backEnd로 연결하는 방법

```js
const socket = new WebSocket("ws://localhost:3000")
```

연결된 결과

![image](https://i.imgur.com/GxwPpuz.png)

- 핸드폰에서도 접속할 수 있도록 uri 변경

```js
const socket = new WebSocket(`ws://${window.location.host}`)
```

-> 여기서 생성된 socket이 frontend와 real-time(실시간)으로 소통할 수 있다.

- **server.js(backend)**의 socket은 연결된 브라우저를 의미한다. (frontend로 메세지를 보낼 수 있고 메세지를 받을 수도 있다.)

- **app.js(frontend)**의 socket은 서버로의 연결을 뜻한다. (backend로 메세지를 보낼 수 있고 메세지를 받을 수도 있다.)



## #1.4 WebSocket Message

- Connection을 해서 Socket을 알 수 있고, 브라우저에 메세지를 보낼 수 있다.

-> (서버가 아닌)socket에 있는 메서드를 사용

```js
wss.on("connection", socket => {
    socket.send("hello!!!");
});
```
-> socket으로 직접적인 연결을 제공해주고 data를 전달해준다.

-> frontEnd에서 message 받기를 처리해주어야 한다. 여기서 message는 event이다.

```js
socket.addEventListener("open", () => {
    console.log("Connected to Server ✅");
});
```

-> socket이 connection을 open했을 때 발생한다.

-> socket에 event를 추가해보자.

```js
socket.addEventListener("message", (message) => {
    console.log("Just got this: ", message, " from the Server");
})
```

-> Server와 Connection이 끊겼을 때 이벤트도 추가해보자.
```js
socket.addEventListener("close", () => {
    console.log("Disconnected from Server ❌");
});
```

-> 연결된 화면 결과

![image](https://i.imgur.com/yK92S3z.png)

-> 이제 backEnd의 socket에서 event를 listen해보자.

```js
socket.on("close", () => console.log("Disconnected from the Browser ❌"));
```

-> 브라우저 탭을 닫은 결과 : 서버가 오프라인이 되면 브라우저에게 알려준다.

![image](https://i.imgur.com/hbGVlcE.png)

-> frontEnd에서 backEnd로 메세지를 보내보자.

frontEnd

```js
setTimeout(() => {
    socket.send("hello from the browser!");
}, 10000);
```

backEnd

```js
socket.on("message", (message) => {
        console.log(message.toString('utf8'));
    });
```
(브라우저가 서버에 메세지를 보냈을 때를 위한 Listener)

-> 메세지 전송 결과 

![image](https://i.imgur.com/7Zr3sgF.png)

- 이 코드는 backEnd와 연결된 각 브라우저에 대해 작동할 것이다.
- 이후 frontEnd와 backEnd에서 메세지 주고받기가 가능하다.

![image](https://i.imgur.com/YvPDIki.png)



## #1.5 Recap

- 우리는 frontEnd와 backEnd 두 곳에서 event를 listen하고 있다.
- backEnd에서는 websocket server를 만들었다.
- 그리고 connection이라는 event를 listen하고 있다.
- connection이 연결되면 socket에서 누가 연결했는지 확인할 수 있다.
- JavaScript는 방금 연결된 socket을 채워줄 것이다.
- 브라우저가 연결되었고, 브라우저마다 연결된 socket에서 event를 listen할 수 있다.
- socket이 connection을 종료시키면 예를 들어, 브라우저의 탭을 닫거나, 컴퓨터가 잠자기 모드에 들어가면 우리는 close해줄 것이다.
- 또한, 우리는 특정 socket에서 메세지를 기다리고 있다.
- 메세지를 기다리기 위해 event listener를 등록해준다. (서버에는 등록하지 않음 - 이 event listener는 backEnd와 연결한 각 브라우저를 위한 것이기 때문)
- ```wss```는 서버 전체를 위한 것이다.
- ```socket.on message```는 특정 socket에서 message를 받았을 때 발생한다.


---

1. event listener를 추가해주었고, message를 브라우저로 전달했다.
2. 브라우저에서는 backEnd와 connection을 열어주고 있다.
3. 그리고 다시 event listener를 등록해주었다.
4. frontEnd -> backEnd / backEnd -> frontEnd로 message를 보낼 수 있다.
5. frontEnd와 backEnd가 양방향 연결이 되었다.
6. 마지막으로 형식이 같으니 frontEnd와 backEnd 파일 구분 잘 하자!



## #1.6 Chat Completed

- html에서 form을 통해 message를 server에 전달.
- server에서 event Listener를 통해 message 전달 받음.

->frontEnd의 form에서 backEnd로 메세지를 전송하고 있다.

![image](https://i.imgur.com/YwxGyDw.png)

- hello를 쓰고 버튼을 클릭하면 텍스트박스가 비워진다.

![image](https://i.imgur.com/6pYSEVq.png)
![image](https://i.imgur.com/ImpNlO8.png)

- backEnd로 메세지 전송 결과

![image](https://i.imgur.com/emNAzeH.png)

- backEnd에서 frontEnd로 같은 메세지를 다시 돌려준다.

![image](https://i.imgur.com/PjNguq4.png)

- backEnd로부터 메세지를 받았다. 

![image](https://i.imgur.com/BEAILHG.png)

- 한 개의 서버가 서로 다른 두 브라우저로 부터 메세지를 받고 있다. (그렇지만 아직 chrome에 연결을 안 해줬기 때문에 다른 브라우저는 서로 메세지를 주고 받지 못한다.)

- 코드가 두 개의 브라우저에 연결되는데 몇명인지는 모른다. 그래서 fake database를 만들어줄 것이다.

- 누군가 우리 서버에 연결하면, 그 connection을 sockets라는 배열 변수에 넣어줄 것이다.

- 이렇게 하면 받은 메세지를 다른 모든 socket에게 전달해 줄 수 있다.

![image](https://i.imgur.com/3MA9ArD.png)

- 실행 결과 화면

![image](https://i.imgur.com/9QBsrs3.png)

