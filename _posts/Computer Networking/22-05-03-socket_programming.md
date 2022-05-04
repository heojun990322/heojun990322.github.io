---
toc: true
toc_sticky: true
categories:
- Computer Networking
title: "파이썬을 이용한 TCP 소켓 프로그래밍"
---

## 대소문자 변환

### 개요
클라이언트가 서버에게 대문자나 소문자로 이루어진 문장을 전송하면 서버는 대문자는 소문자로, 소문자는 대문자로 문장을 수정하여 클라이언트에게 전송한다. 만약 문장에 대소문자가 섞여있는 경우 문장을 소문자로 바꿀 것인지 대문자로 바꿀 것인지 선택할 수 있다.

### Client.py

- **소스 코드**<br>
    ```python
    from socket import *
    serverName = '127.0.0.1' # 서버 ip 주소, 루프백 사용
    serverPort = 12000 # 서버 포트 번호
    clientSocket = socket(AF_INET, SOCK_STREAM) # clientSocket이라고 하는 클라이언트의 소켓 생성, AF_INET -> IPv4사용, SOCK_STREAM -> TCP 소켓
    clientSocket.connect((serverName, serverPort)) # 클라이언트와 서버 간에 TCP 연결을 시도, 세 방향 핸드셰이크 수행
    sentence = input("Input sentence : ") # 문장 입력

    # modeNum의 1번은 대문자 변환, 2번은 소문자 변환
    if sentence.isupper(): # 문장이 전부 대문자인 경우
        modeNum = '2'
    elif sentence.islower(): # 문장이 전부 소문자인 경우
        modeNum = '1'
    else: # 문장이 대소문자가 섞여있는 경우
        modeNum = input("1. To upper case\n2. To Lower case\nInput number : ") 

    # 문장과 modeNum을 클라이언트 소켓을 통해 TCP 연결로 보낸다
    clientSocket.send(sentence.encode())
    clientSocket.send(modeNum.encode())
    modifiedSentence = clientSocket.recv(1024) # 수정된 문자열을 서버로부터 수신하여 저장
    print("Form Server :", modifiedSentence.decode()) # 수정된 문자열 출력
    clientSocket.close() # 클라이언트 소켓을 닫는다
    ```

- **코드 분석**<br>

    1. 문장 입력 까지<br>
        ```python
        from socket import *
        serverName = '127.0.0.1' # 서버 ip 주소, 루프백 사용
        serverPort = 12000 # 서버 포트 번호
        clientSocket = socket(AF_INET, SOCK_STREAM) # clientSocket이라고 하는 클라이언트의 소켓 생성, AF_INET -> IPv4사용, SOCK_STREAM -> TCP 소켓
        clientSocket.connect((serverName, serverPort)) # 클라이언트와 서버 간에 TCP 연결을 시도, 세 방향 핸드셰이크 수행
        sentence = input("Input sentence : ") # 문장 입력     
        ```

        clientSocket에 socket을 생성한다. 이때 파라미터 값으로 AF_INET, SOCK_STREAM을 줬는데 IPv4를 사용하는 TCP 소켓이라는 의미이다. connect method를 통해 서버와 TCP 연결을 시도한다. serverName과 serverPort로 구성된 튜플을 파라미터로 전달한다. connect이후 3-way-handshake가 진행된 후 서버와 TCP 연결이 생성된다. sentence에 문장을 입력받는다.

    2. 문장 파악<br>
        ```python
        # modeNum의 1번은 대문자 변환, 2번은 소문자 변환
        if sentence.isupper(): # 문장이 전부 대문자인 경우
            modeNum = '2'
        elif sentence.islower(): # 문장이 전부 소문자인 경우
            modeNum = '1'
        else: # 문장이 대소문자가 섞여있는 경우
            modeNum = input("1. To upper case\n2. To Lower case\nInput number : ")    
        ```

        문장이 전부 대문자면 modeNum을 2로 설정, 전부 소문자면 1을 설정하고 대소문자가 섞여있으면 대소문자 중 무엇으로 변환할지 사용사가 modeNum을 설정한다.

    3. 소켓을 닫을 때 까지<br>
        ```python
        # 문장과 modeNum을 클라이언트 소켓을 통해 TCP 연결로 보낸다
        clientSocket.send(sentence.encode())
        clientSocket.send(modeNum.encode())
        modifiedSentence = clientSocket.recv(1024) # 수정된 문자열을 서버로부터 수신하여 저장
        print("Form Server :", modifiedSentence.decode()) # 수정된 문자열 출력
        clientSocket.close() # 클라이언트 소켓을 닫는다
        ```

        문장과 modeNum을 byte 형태로 encode하여 send method로 서버에 전송한다. 서버가 문장을 변환하고 클라이언트에게 보내면 소켓의 recv method로 받는다. 받은 문장은 modifiedSentence에 저장되고 decode하여 출력 후 close method로 clientSocket을 닫는다. 

### Server.py

- **소스 코드**<br>
    ```python
    from socket import *
    serverName = '127.0.0.1' # 서버 ip 주소, 루프백 사용
    serverPort = 12000 # 서버 포트 번호
    serverSocket = socket(AF_INET, SOCK_STREAM) # serverSocket이라고 하는 서버의 소켓 생성, AF_INET -> IPv4사용, SOCK_STREAM -> TCP 소켓
    serverSocket.bind((serverName, serverPort)) # 소켓을 포트에 맵핑
    serverSocket.listen(1) # 서버가 클라이언트로부터의 TCP 연결 요청을 듣도록 한다, 파라미터는 연결의 최대 수
    print("The server is ready to receive")

    while True:
        connectionSocket, addr = serverSocket.accept() # 클라이언트의 연결 요청이 들어오면 accept를 호출해서 클라이언트와 연결된 소켓을 생성하고 주소 정보를 리턴
        # 문장과 modeNum 수신
        sentence = connectionSocket.recv(1024).decode()
        modeNum = connectionSocket.recv(1024).decode()

        # modeNum에 따라 문장의 대소문자 변경
        if modeNum == '1':
            modifiedSentence = sentence.upper()
        elif modeNum == '2':
            modifiedSentence = sentence.lower()
        else:
            modifiedSentence = "Error"

        connectionSocket.send(modifiedSentence.encode()) # 클라이언트와 연결된 소켓을 통해 TCP 연결로 보낸다
        connectionSocket.close() # 클라이언트 연결 소켓을 닫는다
    ```

- **코드 분석**<br>
    
    1. TCP 요청을 듣기 시작할 때 까지<br>
    ```python
    from socket import *
    serverName = '127.0.0.1' # 서버 ip 주소, 루프백 사용
    serverPort = 12000 # 서버 포트 번호
    serverSocket = socket(AF_INET, SOCK_STREAM) # serverSocket이라고 하는 서버의 소켓 생성, AF_INET -> IPv4사용, SOCK_STREAM -> TCP 소켓
    serverSocket.bind((serverName, serverPort)) # 소켓을 포트에 맵핑
    serverSocket.listen(1) # 서버가 클라이언트로부터의 TCP 연결 요청을 듣도록 한다, 파라미터는 연결의 최대 수
    print("The server is ready to receive")
    ```
    
        serverSocket에 socket을 생성한다. bind method로 serverPort를 serverSocket에 할당한다. 할당된 serverSocket은 listen method로 TCP 연결 요청을 듣기 시작한다.

    2. 클라이언트와 연결된 소켓을 닫을 때 까지<br>
    ```python
    while True:
        connectionSocket, addr = serverSocket.accept() # 클라이언트의 연결 요청이 들어오면 accept를 호출해서 클라이언트와 연결된 소켓을 생성하고 주소 정보를 리턴
        # 문장과 modeNum 수신
        sentence = connectionSocket.recv(1024).decode()
        modeNum = connectionSocket.recv(1024).decode()

        # modeNum에 따라 문장의 대소문자 변경
        if modeNum == '1':
            modifiedSentence = sentence.upper()
        elif modeNum == '2':
            modifiedSentence = sentence.lower()
        else:
            modifiedSentence = "Error"

        connectionSocket.send(modifiedSentence.encode()) # 클라이언트와 연결된 소켓을 통해 TCP 연결로 보낸다
        connectionSocket.close() # 클라이언트 연결 소켓을 닫는다
    ```

        serverSocket이 TCP 연결 요청을 듣고 있다가 요청이 오면 accept method로 클라이언트와 TCP 연결된 소켓을 connectionSocket에 생성한다. 클라이언트가 보낸 sentence와 modeNum을 connectionSocket을 통해 수신하고 modeNum 번호에 따라 sentence의 대소문자를 변경한다. 수정된 sentence를 클라이언트에게 보내고 connectionSocket을 close method로 닫는다.
    

### 실행 결과

- **Server.py**<br>
    ![exe01](/assets/img/Computer%20Networking/22-05-03-socket_programming/exe01.PNG)<br>

    서버를 실행시키면 서버는 TCP 연결 요청을 받을 준비를 하고 받을 준비가 됐을 때 위와 같이 출력한다.

- **Client.py**<br>

    1. 소문자 문자열 입력<br>
        ![exe02](/assets/img/Computer%20Networking/22-05-03-socket_programming/exe02.PNG)<br>

        소문자로 이루어진 'hi'라는 문자열을 입력하면 서버가 대문자로 수정한 'HI'를 전송한다.

    2. 대문자 문자열 입력<br>
        ![exe03](/assets/img/Computer%20Networking/22-05-03-socket_programming/exe03.PNG)<br>

        대문자로 이루어진 'HI'라는 문자열을 입력하면 서버가 소문자로 수정한 'hi'를 전송한다.

    3. 대소문자 문자열 입력<br>
        ![exe04](/assets/img/Computer%20Networking/22-05-03-socket_programming/exe04.PNG)<br>

        대소문자로 이루어진 문자열 'Hi'를 입력하면 대소문자 중 무엇으로 변환할지 선택한다. 1번을 선택한 경우 위와 같이 서버는 대문자로 수정하여 'HI'를 전송한다.<br>

        ![exe05](/assets/img/Computer%20Networking/22-05-03-socket_programming/exe05.PNG)<br>

        2번을 선택한 경우에는 서버는 소문자로 수정하여 'hi'를 전송한다.

## 로그인

### 개요

### Client.py

### Server.py

### 실행 결과