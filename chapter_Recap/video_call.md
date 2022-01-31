# #3 VIDEO CALL

## #3.0 User Video

- 초기 설정
  1. 먼저 유저로부터 비디오를 가져와 화면에 비디오를 보여줘야 한다.
  2. 마이크를 음소거 및 음소거 해제하는 버튼과 카메라를 껐다 켰다 하는 버튼을 만들어야 한다. 
  3. 전면과 후면 간의 카메라를 전환할 수 있다면 핸드폰에서도 이 앱을 사용할 수 있다.

```home.pug```
```js
video#myFace(autoplay, playsinline, width="400", height="400")
```
-> ```autoplay``` : 자동 재생
-> ```playsinline``` : 모바일 브라우저가 필요로 하는 property. 모바일 기기로 비디오를 재생할 때, 전체화면이 되는 것을 방지해준다. 대신 오직 웹 사이트에서만 실행이 된다.

```app.js```
```js
let myStream;

async function getMedia() {
    try{
        myStream = await navigator.mediaDevices.getUserMedia(
          {
              audio: true,
              video: true,
          });
          myFace.srcObject = myStream;
    } catch(e) {
        console.log(e);
    }
}
getMedia();
```

-> ```stream``` : 비디오와 오디오가 결한된 것.

-> ```getMedia()``` : 오디오 및 비디오 실행 코드

```js
const muteBtn = document.getElementById("mute");
const cameraBtn = document.getElementById("camera");

let muted = false;
let cameraOff = false;
  
function handleMuteClick() {
    if(!muted){
        muteBtn.innerText = "Unmute";
        muted = true;
    }else {
        muteBtn.innerText = "Mute";
        muted = false;
    }
}

function handleCameraClick() {
    if(cameraOff){
        cameraBtn.innerText = "Turn Camera Off";
        cameraOff = false;
    }else {
        cameraBtn.innerText = "Turn Camera On";
        cameraOff = true;
    }
}

muteBtn.addEventListener("click", handleMuteClick);
cameraBtn.addEventListener("click", handleCameraClick);
```

-> 버튼 클릭시 on -> off / mute -> unmute로 변경.

## #3.1 Call Controls

- Stream은 track이라는 것을 제공해준다. 비디오나 오디오, 자막은 하나의 track이 될 수 있다. 

![image](https://i.imgur.com/pcq7INq.png)

- 버튼 클릭시 audio mute/unmute

```js
myStream.getAudioTracks().forEach(track => (track.enabled = !track.enabled));
```

- 버튼 클릭시 vedio camera on/off

```js
myStream.getVideoTracks().forEach(track => (track.enabled = !track.enabled));
```

- getUserMedia() : 유저의 카메라와 오디오를 가져온다.

- enumerateDevices() : 모든 장치와 미디어 장치를 알려준다. (컴퓨터에 연결되거나 모바일이 가지고 있는 장치들)

```js
async function getCameras() {
    try {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const cameras = devices.filter(device => device.kind === "videoinput");
        console.log(cameras);
    } catch (e) {
        console.log(e);
    }
}
```

-> 모든 장치들 중 camera 2개만 적용 = 유저의 비디오에 접근.

- camera의 deviceId와 label을 알아내서 select박스로 option을 추가해보자.

```js
const cameraSelect = document.getElementById("cameras");

const cameras = devices.filter(device => device.kind === "videoinput");
        cameras.forEach(camera => {
            const option = document.createElement("option");
            option.value = camera.deviceId;
            option.innerText = camera.label;
            cameraSelect.appendChild(option);
        });
```

![image](https://i.imgur.com/SNTsQNS.png)

-> 물론 우리에게 중요한 것은 label이 아닌 deviceId이다.



## #3.2 Camera Switch

- 카메라 전환을 구현
  1. select에 input이 변경되었는지 감지하기. -> addEventListener() 활용
  2. 이후 video를 다시 시작하게 만들기. -> getMedia() 재호출
  3. getMedia function으로 사용하려는 특정 카메라 Id를 전송한다.
  4. 처음에 deviceId가 없이 getMedia()를 호출했을 때를 대비하여  constraints를 설정해준다. 
  5. deviceID의 유무에 따라 getMedia() 설정을 달리 해준다.
  6. cameras는 한번만 불러오면 된다.
  7. 현재 사용하고 있는 카메라가 select에 보여지도록 한다. -> myStream.getVideoTracks() 활용



- 3번 코드

```js
async function getMedia(deviceId) {
    try{
        myStream = await navigator.mediaDevices.getUserMedia(
          {
              audio: true,
              video: true,
          });
          myFace.srcObject = myStream;
          await getCameras();
    } catch(e) {
        console.log(e);
    }
}
getMedia();

async function handleCameraChange() {
    await getMedia(cameraSelect.value);
}

cameraSelect.addEventListener("input", handleCameraChange);
```

- 4/5번 코드
```js
const initialConstraints = {
        audio: true,
        video: { facingMode : "user"},
    };

const cameraConstraints = {
        audio : true,
        video: {deviceId: { exact: deviceId }},
    };

try{
        myStream = await navigator.mediaDevices.getUserMedia(
            deviceId ? cameraConstraints : initialConstraints
        );
          myFace.srcObject = myStream;
          if(!deviceId) {
            await getCameras();
          }
    } catch(e) {
        console.log(e);
    }
```

-> initialConstraints는 deviceId가 없을 때 실행된다. cameras를 만들기 전.

-> cameraConstraints는 deviceId가 있을 때 실행된다.

-> 처음 getMedia를 불러올 때만 getCameras()가 실행된다.

- 6/7번 코드

```js
async function getCameras() {
    try {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const cameras = devices.filter(device => device.kind === "videoinput");
        const currentCamera = myStream.getVideoTracks()[0];
        cameras.forEach(camera => {
            const option = document.createElement("option");
            option.value = camera.deviceId;
            option.innerText = camera.label;
            if(currentCamera.label === camera.label) {
                option.selected = true;
            }
            cameraSelect.appendChild(option);
        });
    } catch (e) {
        console.log(e);
    }
}

```
->  myStream.getVideoTracks() 활용하여 현재 사용하는 카메라를 select에 띄운다.

-> stream의 현재 카메라와 paint할 때의 카메라 option을 가져온다.

-> 만약 카메라 option이 현재 선택된 카메라와 같은 label을 가지고 있다면 이것이 우리가 사용하고 있는 카메라라는 것을 알 수 있다.



## #3.3 Introduction to WebRTC

### WebRTC

- web Real-Time Communication

- 실시간 커뮤니케이션을 가능하게 해주는 기술

- peer to peer : video or audio, text가 서버로 가지 않음을 뜻함. 서버가 필요 없다. -> 실시간이 속도가 엄청 빠른 이유.

- webSocket에서 우리는 메세지를 보낼 때 '너'에게 바로 보내는 것이 아니라 '서버'에 메세지를 보내는 거고 그 '서버'가 '너'한테 메세지를 보내는 것이었다. 이것은 peer to peer가 아니다. 

- 내 브라우저가 상대 브라우저에 직접 연결이 된다.

- signaling에서 서버가 필요하다. 상대 브라우저가 어디에 있는지 알기 위해서 서버를 사용한다. 우리 브라우저로 하여금 서버가 상대가 어디에 있는지 알게 해주는 것이다. 

- 브라우저는 서버한테 settings와 configuration, 브라우저의 위치, 방화벽이나 라우터가 있는지 등등의 정보만 전달한다. 그리고 서버는 그 정보를 다른 브라우저에 전달한다. 이후 브라우저는 서로를 찾을 수 있게 된다.


## #3.4 Rooms

- 방 이름 작성후 입장하여 비디오 및 오디오를 활성해보자.

```home.pug```

```js
div#welcome
                form
                    input(placeholder="room name", required, type="text")
                    button Enter room
            div#call
                div#myStream
                    video#myFace(autoplay, playsinline, width="400", height="400")
                    button#mute Mute
                    button#camera Turn Camera Off
                    select#cameras
```

```app.js```

```js
// Welcome Form (join a room)


const welcome = document.getElementById("welcome");
const welcomeForm = welcome.querySelector("form");

function startMedia() {
    welcome.hidden = true;
    call.hidden = false;
    getMedia();
}

function handleWelcomeSubmit(event) {
    event.preventDefault();
    const input = welcomeForm.querySelector("input");
    socket.emit("join_room", input.value, startMedia);
    roomName = input.value;
    input.value="";
}

welcomeForm.addEventListener("submit", handleWelcomeSubmit);


// Socket Code


socket.on("welcome", () => {
    console.log("someone joined");
})
```

```server.js```

```js

wsServer.on("connection", (socket) => {
    socket.on("join_room", (roomName, done) => {
        socket.join(roomName);
        done();
        socket.to(roomName).emit("welcome");
    })
})
```

- 실행 결과 화면

![image](https://i.imgur.com/IxCfHVN.png)



## #3.5 Offers

- 두 서버를 연결시키기 위한 Peer A의 Offers
  1. peerConnection을 두개의 브라우저에 만든다.
  2. addStream을 사용한다. 사용하는 카메라에서 오는 이 stream을 가져다가 이 stream의 데이터를 가져다가 연결을 만들 것이다. 우리는 영상과 오디오를 연결을 통해 전달할 것이다. 그러므로 peer-to-peer 연결 안에 영상과 오디오를 집어넣어야 한다.
  3. peer A는 createOffer를 생성하고 peer B는 createAnswer를 생성한다.
    - 방에 참가하면 알림을 받는 브라우저가 peer A
  4. setLocalDescription을 만든다. 우리가 만든 offer로 연결을 구성해야 한다. 


- 1/2/3번 실행 결과 화면 (offer 확인)

![image](https://i.imgur.com/wpnvqvc.png)

```js
socket.on("welcome", async () => {
    const offer = await myPeerConnection.createOffer();
    myPeerConnection.setLocalDescription(offer);
    socket.emit("offer", offer, roomName);
})
```
-> 이 코드는 peer A 브라우저에서만 실행된다. 알림을 받는 건 peer A 뿐이기 때문.

```js
socket.on("offer", (offer) => {
    console.log(offer);
})
```
-> 이 코드는 peer B 브라우저에서만 실행된다. offer도 포함되어 있음.

- 3/4번 실행 결과 화면

![image](https://i.imgur.com/egjtuWz.png)

-> peer B 브라우저에 offer이 전달되었다.

-> 이것이 바로 Signaling process이다. 즉, offer를 주고 받기 위해서는 서버가 필요하다. offer가 주고 받아진 순간, 우리는 직접적으로 대화를 할 수 있다. 



## #3.6 Answers

- Peer A에서 offer를 만들었고, 그 다음 setLocalDescription을 해줬다. 이는 Peer A한테 이 description을 알려줬다는 것을 의미한다. Peer A를 위한 localDescription(**offer**)을 보내고 Peer B부분의 RemoteDescription에서 그 offer를 받는다. 즉, 멀리 떨어진 peer의 description을 셋팅을 한다. 

- peer A 브라우저에서 offer를 만들었을 때, LocalDescription을 set한다. peer B 브라우저에서 offer를 받으면, setRemoteDescription을 하고 그리고 같은 함수에서 answer를 만들고 LocalDescription을 set한다. 그러곤 answer를 다시 peer A 브라우저로 돌려보낸다. 이후 peer A 브라우저가 setRemoteDescription을 할 수 있다. 

- Peer B의 Answers ('#3.5'에 이어서) 
  1. setRemoteDescription을 만든다. 
  2. offer의 전송이 socketIO로 빠르기 때문에 myPeerConnection이 안받아질 수 있다. handleWelcomeSubmit에서 미리 getMedia()를 불러오자.
  3. answer를 받는다.
  4. peer A로 answer를 보낸다.
  5. 여기까지 실행이 됐다면 두 브라우저는 모두 LocalDescription과 RemoteDescription을 가지게 된다.

- 1/2/3번 실행 결과 화면

![image](https://i.imgur.com/4soHsxg.png)

### 정리
1. peer A 브라우저에서 offer를 만든다.

2. peer B 가 방에 참가했을 때 offer를 만든다. 이 offer로 setLocalDescription을 한다.

3. peer B 가 들어오면 peer A가 이 코드를 실행한다.

```app.js```

```js
socket.on("welcome", async () => {
    const offer = await myPeerConnection.createOffer();
    myPeerConnection.setLocalDescription(offer);
    console.log("sent the offer");
    socket.emit("offer", offer, roomName);
})
```

4. 서버에 보내면 peer B 브라우저가 그 offer를 받고 setRemoteDescription을 한다.

```app.js```

```js
socket.on("offer", async(offer) => {
    myPeerConnection.setRemoteDescription(offer);
    const answer = await myPeerConnection.createAnswer();
    myPeerConnection.setLocalDescription(answer);
    socket.emit("answer", answer, roomName);
})
```

5. 이후 peer B 브라우저는 answer를 생성한다.
```js
const answer = await myPeerConnection.createAnswer();
```

6. peer B는 그 answer로 setLocalDescription을 한다. ( getUserMedia랑 addStream은 우리가 이미 peer B가 방에 들어왔을 때 그것들을 실행했기 때문에 생략한다. )

7. answer을 보낸 후 그 asnwer은 다시 slignaling server 즉, Socket.IO로 돌아간다. 

8. peer A 브라우저에서 그 함수를 받았을 때, 이 코드를 실행해준다.

```js
socket.on("answer", (answer) => {
    myPeerConnection.setRemoteDescription(answer);
})
```


## #3.7 IceCendidate

### IceCandidate
> Internet Connectivity Establishment(인터넷 연결 생성)

: webRTC에 필요한 프로토콜들을 의미한다. 멀리 떨어진 장치와 소통할 수 있게 하기 위함. **즉, 브라우저가 서로 소통할 수 있게 해주는 방법이다.** 어떤 소통 방법이 가장 좋을 것인지를 제안할 때 쓰인다. 다수의 candidate(후보)들이 각각의 연결에서 제안되고 그리고 그들은 서로의 동의 하에 하나를 선택한다. 그리고 그것을 소통 방식에 사용한다. (answer와 다른 모든 것들을 얻고 난 뒤에 일어남.)

- offer와 answer의 핑퐁을 모두 끝냈을 때, peer-to-peer 연결의 양쪽에서 icecandidate라는 이벤트를 실행하기 시작한다.

- 실행 결과 화면

![image](https://i.imgur.com/MQtgX9P.png)

- 각 브라우저에 candidate를 보낸다.

```app.js```

```js
socket.on("ice", (ice) => {
    console.log("received candidate");
    myPeerConnection.addIceCandidate(ice);
});

function handleIce(data) {
    console.log("sent candidate");
    socket.emit("ice", data.candidate, roomName);
}
```

```server.js```

```js
socket.on("ice", (ice, roomName) => {
        socket.to(roomName).emit("ice", ice);
    })
```

- 실행 결과 화면

![image](https://i.imgur.com/ozSQ1NV.png)

- add Stream event 등록 (각 브라우저에 상대 stream 출력)

```app.js```

```js
function handleAddStream(data) {
    console.log("got an stream from my peer");
    console.log("Peer's Stream: ", data.stream);
    console.log("My Stream: ", myStream);
}
```

- 실행 결과 화면

![image](https://i.imgur.com/1Zhz85d.png)

- 서로의 stream을 받고 비디오를 하나 더 생성

```home.pug```

```js
 video#peerFace(autoplay, playsinline, width="400", height="400")
```

```app.js```

```js
function handleAddStream(data) {
    const peerFace = document.getElementById("peerFace");
    peerFace.srcObject = data.stream;
}
```

- 실행 결과 화면

![image](https://i.imgur.com/z2hiszB.jpg)

-> 문제점 발생 : 카메라를 바꾸게 되면 stream이 끝나버려서 다른 브라우저에서 적용이 되지 않는다. (더이상 answer를 받지 않음.)



## #3.8 Senders

- 카메라 바꾸는 걸 다루는 function을 다뤄보자.

- 카메라를 바꿀 때마다, makeConnection()에서 새로운 stream을 만들기 때문에 다른 브라우저에서 적용이 안된다. 이 문제를 해결해보자.

```js
async function handleCameraChange() {
    await getMedia(cameraSelect.value);
    if(myPeerConnection) {
        console.log(myPeerConnection.getSenders());
    }
}
```

- 카메라를 바꾼 후 video를 보내는 sender 실행 결과 화면

![image](https://i.imgur.com/WJP7fjd.png)

- 카메라를 바꿀 때 stream을 새로 바꿔버리지만 user한테 보내는 track은 바꾸지 않는다. 이 문제를 해결하기 위해 Real Time Communication 연결의 track을 바꿔보자. (kind:"video"를 가진 sender를 찾아서 getSender해주기)

- **Sender** : Sender는 우리의 peer로 보내진 media stream track을 컨트롤 할 수 있게 해준다. (우리의 stream을 음소거(mute)할 수도 있고, peer-to-peer connection의 track을 변경할 수도 있다.)

- getMedia(deviceId) function에서 다른 deviceId로 새로운 stream을 만들고 handleCameraChange()에서 video device의 새로운 id로 또다른 stream을 생성했다. 이 뜻은 이 부분 이후로 만약 내가 video track을 받으면 새로 선택한 장치로 업데이트 된 video track을 받는 것이다. 그러므로 handleCameraChange()에서 videoTrack을 받을 수 있다. 그리고 sender를 그 video track으로 바꿔준다. 

```app.js```

```js
async function handleCameraChange() {
    await getMedia(cameraSelect.value);
    if(myPeerConnection) {
        const videoTrack = myStream.getVideoTracks()[0]; 
        const videoSender = myPeerConnection
        .getSenders()
        .find((sender) => sender.track.kind === "video");
        videoSender.replaceTrack(videoTrack);
    }
}
```

-> 우리는 현재 2개의 track을 가지고 있다. 하나는 나 자신을 위한 myStream이다. 동시에 다른 peer 브라우저로 데이터를 보낼 때가 Sender를 사용하는 때이다. Sender는 다른 브라우저로 보내진 비디오와 오디오 데이터를 컨트롤하는 방법이다. 

- 실행 결과 화면

![image](https://i.imgur.com/2RITbrx.jpg)

-> 문제점 발생 : 이 웹사이트를 폰으로 접속하면 작동하지 않는다. 

- 문제점을 해결하기 위해 local tunnel을 설치하자.

``` npm i localtunnel```

- local tunnel은 서버를 전세계와 공유하게 해준다. (일시적으로만 무료임.) local tunnel을 사용하면 우리 서버의 url을 생성할 수 있다. 즉, 폰으로 그 url에 접속해서 테스트해 볼 수 있다. 

- 글로벌로 설치해보자.

```npm i -g localtunnel```

- localtunnel이 글로벌로 설치되면 lt를 사용해서 licaltunnel을 호출한다. 이제 커맨드 사용이 가능하다. lt는 local tunnel를 뜻한다. 전세계와 특정 포트를 공유할 수 있다. 커맨드를 실행해보자.

```lt --port 3000```

- url 실행 결과 화면

<img src="https://i.imgur.com/L5G8W3K.png" width="400" height="700"/>

-> 해당 tunnel 웹사이트라고 알려주는 화면. continu 클릭.

-> 문제점 발생 : localtunnel url로 접속하니 504 Gateway Time-Out이 떴다. 

-> 문제점 해결 : lt 프로세스 실행을 위해 node 서버를 실행종료 시켜서 발생한 문제였다. lt 프로세스를 백그라운드에서 실행시키고 node 서버를 재실행 시키니 문제가 해결되었다. 

<img src="https://i.imgur.com/KMibsC6.png" width="700" height="300"/>

- 핸드폰에서 room 입장 후 화면

<img src="https://i.imgur.com/OfdA9hV.png" width="610" height="720"/> 

<img src="https://i.imgur.com/7bz3exj.png" width="300" height="600"/> <img src="https://i.imgur.com/WFqvp5L.jpg" width="300" height="600"/>


## #3.9 STUN

- 이전 #3.8 실행 결과 화면에서 문제점을 발견했다.

- 문제점 발생 : 컴퓨터랑 폰이 같은 wifi에 있지 않으면 에러가 발생. STUN 서버가 없기 때문이다.

- **STUN 서버** : STUN 서버는 컴퓨터가 공용 IP주소를 찾게 해준다. 어떤 것을 request 하면 인터넷에서 사용자가 누군지를 알려주는 서버이다.

- 장치는 공용주소를 알아야 다른 네트워크에 있는 장치들이 서로를 찾을 수 있게 된다. 

- 문제점 해결 : 사용하고자 하는 STUN 서버의 리스트를 추가해주자. 전문적으로 하려면 STUN 서버를 직접 만들어야 하지만 우리는 구글이 무료로 제공해주는 서버를 사용하도록 하자. 

```js
myPeerConnection = new RTCPeerConnection({
        iceServers: [
            {
                urls: [
                    "stun:stun.l.google.com:19302",
                    "stun:stun1.l.google.com:19302",
                    "stun:stun2.l.google.com:19302",
                    "stun:stun3.l.google.com:19302",
                    "stun:stun4.l.google.com:19302",
                ]
            }
        ]
    });
```

- 다른 IP(네트워크)에서도 접속이 잘 되는 것을 확인할 수 있다.

<img src="https://i.imgur.com/2ICggM5.png" width="610" height="720"/>
