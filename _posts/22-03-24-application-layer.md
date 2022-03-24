---
toc: true
toc_sticky: true
categories:
- Computer Network
---
현재 학교에서 컴퓨터 네트워크 과목을 수강 중이다. 수업 시간에 배우고 공부한 내용들을 주기적으로 정리할 것 이다. 이 포스트에서는 OSI 7계층 중 제일 위에 있는 Application Layer에 대해 다룰 것 이다.

## Principles Of Network Applications

### Network Applications
- 서로 다른 end system에서 실행 가능하다.
- network를 통해 통신한다.
- network-core devices를 위한 sofrware를 작성할 필요가 없다.

### Application Development

#### Client
- HTML : 웹 사이트의 구조와 틀을 설계할 수 있다.
- CSS : HTML로 만든 문서의 스타일을 지정할 수 있다.
- JavaScript : 웹 페이지를 동적으로 반응할 수 있게 해준다.
- 관련 Framworks : React.js, Vue.js, Angular.js..

#### Server
- Spring Boot : Java를 이용한 프레임워크로 특히 우리나라에서 수요가 많다.
- Express : Javascript를 이용하는 Node.js로 서버 개발을 가능하게 한다.
- Django : Python을 사용하고 기능 탑재가 쉽고 생산성이 좋다.
- Laravel : PHP를 사용하는 MVC 프레임워크이다.
- Rails : Ruby로 개발된 프레임워크이고 생산성이 장점이다.

### Application Architectures

#### Client-Server
![client-server-web](/assets/img/client-server-web.jpg)
- Server
    - Client로부터 온 요청을 처리한다.
    - 다수의 Client를 처리하기 위해 매우 큰 용량과 성능을 가지고 있다.
    - 고정 IP를 소유하고 있다.
    - 클라우드 컴퓨팅을 위해 필수이다.
- Client
    - Server에게 서비스를 요청한다.
    - 유동 IP를 사용한다.
    - 즉시 연결된다.
    - 다른 클라이언트와 서버를 통해 통신한다.
#### P2P

## References
[Client-Server](https://www.google.co.kr/url?sa=i&url=https%3A%2F%2Fwww.router-switch.com%2Ffaq%2Fwhat-are-server-and-client-definition.html&psig=AOvVaw3s2zNBkd3Vllp8GFpb_rJW&ust=1648190652721000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCKCZ0OWS3vYCFQAAAAAdAAAAABAh)
