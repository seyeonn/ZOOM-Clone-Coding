# #0 INTRODUCTION

## #0.1 Requirements

![image](https://i.imgur.com/lHnnmSb.png)

## #0.2 Server Setup
> Babel, Nodemon, Express를 활용해서 NodeJS 프로젝트를 설정

- nodemon 설치

![image](https://i.imgur.com/Gzamspx.png)

- babel, server, nodemon 생성

![image](https://i.imgur.com/7muglOV.png)

- nodemon의 exec키를 이용하여 src/server.js에 대해 babel-node 명령문을 실행시킨다는 의미를 가진다.

![image](https://i.imgur.com/TFViUJk.png)

- dev는 nodemon을 호출하고 nodemon이 호출되면 nodemon이 nodemon.json을 살펴보고 거기있는 코드를 실행한다.

![image](https://i.imgur.com/Kgpavjm.png)

- express import후 npm run dev 실행

![image](https://i.imgur.com/K914pJ3.png)

- 서버 구동

![image](https://i.imgur.com/C9QFlf0.png)

## #0.3 Frontend Setup

- home으로 가면 request, response를 받고 res.render를 한다. (= home을 render한다.)
- express로 할 일은 views를 설정해주고 render를 해준다. 나머지는 websocket에서 실시간으로 일어난다.

![image](https://i.imgur.com/xjo4vDD.png)

- 페이지(home) render.

![image](https://i.imgur.com/xmn2Q0c.png)

- express 설정 완료
- app.use("/public", express.static(__dirname + "/public")); -> public 폴더를 유저에게 공개한다.

![image](https://i.imgur.com/56Sl1Yp.png)
![image](https://i.imgur.com/MnmeIR8.png)
![image](https://i.imgur.com/Z9u1ufb.png)

## #0.4 Recap
> 개발 환경 구축 - Nodemon을 설정하기 위해 nodemon.json을 생성

- ```Nodemon```은 우리의 프로젝트를 살펴보고 변경사항이 있을 시 서버를 재시작해주는 프로그램이다.

- 서버를 재시작하는 대신에 babel-node를 실행하게 되는데 ```Babel```은 우리가 작성한 코드를 일반 NodeJS 코드로 컴파일 해주는데 그 작업을 ```src/server.js```에 해준다.

- ```server.js```파일에서는 ```express```를 import하고, ```express``` 어플리케이션을 구성하고, ```view engine```을 ```Pug```로 설정하고, ```views``` 디렉토리가 설정되고, 그리고 ```public``` 파일들에 대해서도 똑같은 작업을 해준다.

- ```public``` 파일들은 FrontEnd에서 구동되는 코드이고 중요한 부분이다.

- ```server.js```는 BackEnd에서 구동될 거고, ```app.js```는 FrontEnd에서 구동될 것이다.

- ```express```를 사용한 ```NodeJS``` 설정 완료.
