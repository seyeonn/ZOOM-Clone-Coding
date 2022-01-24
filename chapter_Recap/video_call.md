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

