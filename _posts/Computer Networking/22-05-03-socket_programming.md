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
클라이언트가 아이디와 비밀번호를 입력하면 서버가 로그인 결과를 출력해준다. 구현해 놓은 로그인 관련 기능은 신규 아이디를 등록하는 것과 비밀번호 변경이 있다.

### Client.py

- **소스 코드**

```python
from socket import *
serverAddress = '127.0.0.1' # 서버 주소, 루프백 사용
serverPort = 9999 # 서버 포트 번호
clientSocket = socket(AF_INET, SOCK_STREAM) # 클라이언트 소켓 생성, IPv4 사용, TCP 사용
clientSocket.connect((serverAddress, serverPort)) # 서버 연결 시도
loginMsg = '' # 로그인 관련 메시지

# ID와 PW를 입력 후 서버에 전송
def Login(clientSocket):
    id = input("\nID : ")
    pw = input("PW : ")
    clientSocket.send(id.encode())
    clientSocket.send(pw.encode())
    loginMsg = clientSocket.recv(1024).decode() # Login 결과에 대한 메시지 수신
    return loginMsg

# ID와 PW 등록
def Register(clientSocket):
    id = input("\nNew ID : ")
    pw = input("New PW : ")
    clientSocket.send(id.encode())
    clientSocket.send(pw.encode())
    loginMsg = clientSocket.recv(1024).decode() # Register 결과에 대한 메시지 수신
    return loginMsg

# PW 변경
def ChangePw(clientSocket):
    firstPw = input("\nNew PW : ")
    secondPw = input("Reinput PW : ")
    clientSocket.send(firstPw.encode())
    clientSocket.send(secondPw.encode())
    loginMsg = clientSocket.recv(1024).decode() # ChangePw 결과에 대한 메시지 수신
    return loginMsg    

# 로그인과 등록 중 하나 선택
print("\n1. Login\n2. Register")
num = input("Input number : ")

if num == '1' : # 로그인을 선택한 경우
    loginMsg = 'Login'
    clientSocket.send(loginMsg.encode())
elif num == '2' : # 등록을 선택한 경우
    loginMsg = 'Register'
    clientSocket.send(loginMsg.encode())

while True:
    if loginMsg == 'Success': # 로그인 성공
        print("\nLogin success!")
        clientSocket.close()
        break
    elif loginMsg == 'Login': # 로그인 시도
        loginMsg = Login(clientSocket)
    elif loginMsg == 'Register': # 등록
        loginMsg = Register(clientSocket)

        if loginMsg == 'Register':
            print("\nDuplicate ID.. Reinput different ID")
        elif loginMsg == 'Login':
            print("\nID is registered successfully! Input ID and PW for login")
    elif loginMsg == 'ChangePw': # 비밀번호 변경
        loginMsg = ChangePw(clientSocket)

        if loginMsg == 'ChangePw':
            print("\nNew PW and reinputed PW are different.. Reinput PW")
        elif loginMsg == 'Login':
            print("\nPW is changed successfully! Input ID and PW for login")
    elif loginMsg == 'WrongId' : # 입력한 id가 데이터에 없을 때
        print("\nWrong ID..")
        
        # 재입력과 등록 중 선택
        print("\n1. Reinput\n2. Register")
        num = input("Input num : ")

        if (num == '1'): # 로그인을 선택한 경우
            loginMsg = 'Login'
            clientSocket.send(loginMsg.encode())
        elif (num == '2'): # 등록을 선택한 경우
            loginMsg = 'Register'
            clientSocket.send(loginMsg.encode())
    elif loginMsg == 'WrongPw' : # 입력한 pw가 데이터에 없을 때
        print("\nWrong PW..")

        # 재입력과 비밀번호 변경 중 선택
        print("\n1. Reinput\n2. Change PW")
        num = input("Input num : ")

        if num == '1': # 로그인을 선택한 경우
            loginMsg = 'Login'
            clientSocket.send(loginMsg.encode())
        elif num == '2': # 비밀번호 변경을 선택한 경우
            loginMsg = 'ChangePw'
            clientSocket.send(loginMsg.encode())
```

- **코드 분석**

    1. 서버 연결 까지

        ```python
        from socket import *
        serverAddress = '127.0.0.1' # 서버 주소, 루프백 사용
        serverPort = 9999 # 서버 포트 번호
        clientSocket = socket(AF_INET, SOCK_STREAM) # 클라이언트 소켓 생성, IPv4 사용, TCP 사용
        clientSocket.connect((serverAddress, serverPort)) # 서버 연결 시도
        loginMsg = '' # 로그인 관련 메시지
        ```

        clientSocket에 socket을 생성한다. connect method로 서버에 TCP 연결 요청을 한다. server가 로그인 관련 메시지를 전송하면 클라이언트는 loginMsg에 저장한다. 
    
    2. 로그인과 등록 중 선택

        ```python
        # 로그인과 등록 중 하나 선택
        print("\n1. Login\n2. Register")
        num = input("Input number : ")

        if num == '1' : # 로그인을 선택한 경우
            loginMsg = 'Login'
            clientSocket.send(loginMsg.encode())
        elif num == '2' : # 등록을 선택한 경우
            loginMsg = 'Register'
            clientSocket.send(loginMsg.encode())
        ```

        클라이언트를 실행시키고 처음 사용자가 할 수 있는 것은 로그인을 할 것인지 등록을 할 것인지 선택하는 것 이다. 로그인을 선택한 경우 loginMsg를 Login으로 설정하고 메시지를 서버에게 전송한다. 등록을 선택한 경우 loginMsg를 Register로 설정하고 메시지를 서버에게 전송한다.

    3. 무한 루프

        while 문으로 들어오면 break를 만나기 전까지 무한 loop가 된다. 무한 loop는 loginMsg를 확인하며 메시지에 해당하는 코드를 수행한다.

        ```python
        if loginMsg == 'Success': # 로그인 성공
        print("\nLogin success!")
        clientSocket.close()
        break    
        ```

        loginMsg가 Success인 경우 로그인이 성공 했다는 의미이다. 로그인 성공을 출력하고 clientSocket을 닫고 break문으로 while문을 탈출한다.

        ```python
        elif loginMsg == 'Login': # 로그인 시도
        loginMsg = Login(clientSocket)
        ```

        loginMsg가 Login인 경우 로그인을 시도한다. 로그인시도는 Login 함수를 호출하여 진행하고 loginMsg에 결과에 대한 메시지를 저장한다.

        ```python
        # ID와 PW를 입력 후 서버에 전송
        def Login(clientSocket):
            id = input("\nID : ")
            pw = input("PW : ")
            clientSocket.send(id.encode())
            clientSocket.send(pw.encode())
            loginMsg = clientSocket.recv(1024).decode() # Login 결과에 대한 메시지 수신
            return loginMsg
        ```

        Login 함수이다. id와 pw를 입력받고 서버에 전송한다. 서버로부터 로그인 관련 메시지를 수신한 후 리턴한다. 

        ```python
        elif loginMsg == 'Register': # 등록
            loginMsg = Register(clientSocket)

            if loginMsg == 'Register':
                print("\nDuplicate ID.. Reinput different ID")
            elif loginMsg == 'Login':
                print("\nID is registered successfully! Input ID and PW for login")    
        ```

        loginMsg가 Register면 등록을 진행한다. 등록은 Register 함수를 통해 이루어지고 등록에 대한 메시지를 loginMsg에 저장한다. 이때 loginMsg가 Register면 중복된 ID를 입력한 경우이다. 다음 loop 때 등록을 다시 진행한다. 성공적으로 진행 된 경우 loginMsg는 Login으로 설정된다. 다음 loop 때 login을 진행한다.

        ```python
        # ID와 PW 등록
        def Register(clientSocket):
            id = input("\nNew ID : ")
            pw = input("New PW : ")
            clientSocket.send(id.encode())
            clientSocket.send(pw.encode())
            loginMsg = clientSocket.recv(1024).decode() # Register 결과에 대한 메시지 수신
            return loginMsg
        ```

        Register 함수이다. 등록할 id와 pw를 입력받고 서버에 전송한다. 서버가 등록에 대해 처리한 결과 메시지를 리턴한다.

        ```python
        elif loginMsg == 'ChangePw': # 비밀번호 변경
            loginMsg = ChangePw(clientSocket)

            if loginMsg == 'ChangePw':
                print("\nNew PW and reinputed PW are different.. Reinput PW")
            elif loginMsg == 'Login':
                print("\nPW is changed successfully! Input ID and PW for login")    
        ```

        loginMsg가 ChangePw일 경우 비밀번호를 변경한다. ChangePw 함수를 호출하여 비밀번호를 변경 시도 후 loginMsg을 저장한다. 리턴받은 loginMsg가 ChangPw일 경우 새로 입력한 pw와 확인용 pw가 일치하지 않는 경우이다. 다음 loop때 다시 비밀번호 변경을 시도한다. 비밀번호 변경이 성공한 경우 loginMsg를 Login으로 설정한다. 다음 loop 때 로그인을 시도한다.

        ```python
        # PW 변경
        def ChangePw(clientSocket):
            firstPw = input("\nNew PW : ")
            secondPw = input("Reinput PW : ")
            clientSocket.send(firstPw.encode())
            clientSocket.send(secondPw.encode())
            loginMsg = clientSocket.recv(1024).decode() # ChangePw 결과에 대한 메시지 수신
            return loginMsg    
        ```

        ChangePw 함수이다. pw를 입력받고 확인용 pw를 입력받는다. 두 pw를 서버에 전송 후 처리 결과에 대한 메시지를 return한다.
        
        ```python
        elif loginMsg == 'WrongId' : # 입력한 id가 데이터에 없을 때
            print("\nWrong ID..")
        
            # 재입력과 등록 중 선택
            print("\n1. Reinput\n2. Register")
            num = input("Input num : ")

            if (num == '1'): # 로그인을 선택한 경우
                loginMsg = 'Login'
                clientSocket.send(loginMsg.encode())
            elif (num == '2'): # 등록을 선택한 경우
                loginMsg = 'Register'
                clientSocket.send(loginMsg.encode())  
        ```

        loginMsg가 WrongId면 없는 id를 입력한 경우이다. 사용자는 이 때 로그인을 다시 시도할 것인지 새로운 id를 등록할 것인지 선택할 수 있다. 로그인을 선택한 경우 loginMsg를 Login을 설정하고 등록을 선택한 경우 loginMsg를 Register로 설정한다. 다음 loop 때 해당 loginMsg에 해당하는 코드를 수행할 것 이다.

        ```python
        elif loginMsg == 'WrongPw' : # 입력한 pw가 데이터에 없을 때
            print("\nWrong PW..")

            # 재입력과 비밀번호 변경 중 선택
            print("\n1. Reinput\n2. Change PW")
            num = input("Input num : ")

            if num == '1': # 로그인을 선택한 경우
                loginMsg = 'Login'
                clientSocket.send(loginMsg.encode())
            elif num == '2': # 비밀번호 변경을 선택한 경우
                loginMsg = 'ChangePw'
                clientSocket.send(loginMsg.encode())
        ```

        loginMsg가 WrongPw인 경우 pw를 잘 못 입력한 것이다. 사용자는 pw를 변경할 것인지 다시 로그인을 시도할 것인지 선택할 수 있다. 로그인을 선택한 경우 loginMsg는 Login이 되고 pw 변경을 선택한 경우 loginPw는 ChangePw가 된다.

### Server.py

- **소스 코드**

```python
from multiprocessing.dummy import connection
from socket import *
serverAddress = '127.0.0.1' # 서버 주소, 루프백 사용
serverPort = 9999 # 서버 포트 번호
serverSocket = socket(AF_INET, SOCK_STREAM) # 클라이언트 소켓 생성, IPv4 사용, TCP 사용
serverSocket.bind((serverAddress, serverPort)) # 포트를 서버 소켓에 바인딩
serverSocket.listen(1) # 클라이언트 요청 대기, 최대 연결 가능 수 = 1
print("Server is ready to receive")

code = 0
loginData = {} # id와 pw 정보 데이터
loginCode = {1:'Success', 2:'WrongId', 3:'WrongPw', 4:'Login', 5:'Register', 6:'ChangePw'} # login과 관련된 메시지에 매핑된 코드

def CheckLoginData(connectionSocket, inputId, inputPw): # id와 pw 확인
    try:
        if inputPw == loginData[inputId]: # 입력한 id가 데이터에 존재하고 pw가 일치하는 경우
            connectionSocket.send(loginCode[1].encode())
            return 1
        else: # 입력한 id가 데이터에 존재하지만 pw가 일치하지 않는 경우
            connectionSocket.send(loginCode[3].encode())
            return 3
    except KeyError as e: # 입력한 id가 데이터에 존재하지 않는 경우
        connectionSocket.send(loginCode[2].encode())
        return 2

def Register(connectionSocket, inputId, inputPw): # id와 pw 등록
    if inputId in loginData: # 입력한 id가 있는 id인 경우
        connectionSocket.send(loginCode[5].encode())
        return 5
    
    # 입력한 id가 새로운 id인 경우
    loginData[inputId] = inputPw 
    connectionSocket.send(loginCode[4].encode())
    return 4

def ChangePw(connectionSocket, inputId, inputFirstPw, inputSecondPw): # pw 변경
    if inputFirstPw != inputSecondPw: # 처음 입력한 pw와 확인 pw가 다른 경우
        connectionSocket.send(loginCode[6].encode())
        return 6

    # 처음 입력한 pw와 확인 pw가 같은 경우
    loginData[inputId] = inputFirstPw
    connectionSocket.send(loginCode[4].encode())
    return 4

while True:
    connectionSocket, address = serverSocket.accept() # 연결 요청이 들어오면 연결 소켓을 생성하고 주소를 리턴

    # 로그인과 등록 중 선택
    loginMsg = connectionSocket.recv(1024).decode()
    if loginMsg == 'Login': # 로그인인 경우
        code = 4
    elif loginMsg == 'Register': # 등록인 경우
        code = 5

    while True:
        if loginCode[code] == 'Success': # 로그인 성공한 경우
            connectionSocket.close()
            break
        elif loginCode[code] == 'Login': # 로그인 시도
            inputId = connectionSocket.recv(1024).decode() # 입력한 id 수신
            inputPw = connectionSocket.recv(1024).decode() # 입력한 pw 수신
            code = CheckLoginData(connectionSocket, inputId, inputPw) # id와 pw 확인
        elif loginCode[code] == 'Register': # 등록하는 경우
            inputId = connectionSocket.recv(1024).decode() # 입력한 id 수신
            inputPw = connectionSocket.recv(1024).decode() # 입력한 pw 수신
            code = Register(connectionSocket, inputId, inputPw)
        elif loginCode[code] == 'ChangePw': # 비밀번호를 변경하는 경우
            inputFirstPw = connectionSocket.recv(1024).decode() # 입력한 새로운 pw 수신
            inputSecondPw = connectionSocket.recv(1024).decode() # 입력한 확인 pw 수신
            code = ChangePw(connectionSocket, inputId, inputFirstPw, inputSecondPw)
        elif loginCode[code] == 'WrongId': # id가 잘못된 경우
            loginMsg = connectionSocket.recv(1024).decode()

            if loginMsg == 'Login': # 로그인인 경우
                code = 4
            elif loginMsg == 'Register': # 등록인 경우
                code = 5
        elif loginCode[code] == 'WrongPw' : # PW가 잘못된 경우
            loginMsg = connectionSocket.recv(1024).decode()            
            
            if loginMsg == 'Login': # 로그인인 경우
                code = 4
            elif loginMsg == 'ChangePw': # 비밀번호 변경인 경우
                code = 6
```

- **코드 분석**


### 실행 결과