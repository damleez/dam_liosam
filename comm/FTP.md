## - msg_basic_command.py
### 0. define
- FTP : FTP란 파일 전송 프로토콜(File Transfer Protocol)의 약자로 서버와 클라이언트 사이에서 TCP/IP를 통해 파일을 송수신하기 위해 고안된 프로토콜
- FTP는 다른 프로토콜들과 다르게 포트(port) 번호를 기본 두 개를 사용하도록 제작
  - 첫 번째 포트는 포트번호 21번으로 클라이언트와 서버 사이의 명령, 제어 등을 송수신하는 제어 포트 
  - 그리고 두 번째 포트로는 20번으로 클라이언트와 서버 사이의 직접적인 파일 송 수신을 담당하는 데이터 포트
 
  ![image](https://github.com/damleez/dam_LIO-SAM/assets/108650199/883b64ee-65e0-4f84-8d69-74637b4372a1)

### 1. library insert
```
from ftplib import FTP
```
