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

