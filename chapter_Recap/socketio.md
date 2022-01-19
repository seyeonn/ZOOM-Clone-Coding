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

