PWM control
===
## 1. PWM(Pulse Width Modulation)
```
Pulse : 사각파
Width : 폭
Modulation : 변조
```
- 직역하면 사각파의 폭을 제어한다는 의미

![image](https://user-images.githubusercontent.com/108650199/216231118-39003cd1-3d7b-41e2-b11e-6625276bf6dc.png)
### parameter
- 주파수 Hz : 1초에 진동하는 횟수
- Duty ratio % : 한 주기 안에서 신호가 on 되어 있는 비율
  - 여기서 바로 폭(width)을 변조(modulation) 한다는 의미가 duty ratio를 조절한다는 의미

### PWM control
- PWM 신호는 디지털 신호의 0, 1을 빠르게 반복한 것이라 볼 수 있음
- 즉, HIGH와 LOW를 반복하는 펄스파이며, HIGH가 되는 시간을 조절하여 제어

![image](https://user-images.githubusercontent.com/108650199/216231725-1cf6b0e0-cb9c-47ac-9f86-89ef62ee6ed7.png)

  - 위 그림은 duty ratio를 조절한다고 표현

- 제어도 마찬가지
- 모터의 속도는 아날로그이지만, 마이크로프로세서는 디지털 출력이 용이하므로 PWM 제어를 사용
- 예를 들어, 모터의 속도를 빠르게 하려면 duty ratio를 높이고, 속도를 늦추려면 duty ratio를 낮추면 됩

![image](https://user-images.githubusercontent.com/108650199/216231962-b9368b08-ffed-4b25-aa2d-f4dfa6ff6ba5.png)

## 2. sierra drone PWM control
### 2.1 pixhawk

![123](https://user-images.githubusercontent.com/108650199/216232265-eedae369-a0b3-45c7-a20f-38578b35970d.png)

#### RC control in propeller
- 빨간 네모에서 왼쪽에 있는 부분이 drone 프로펠러의 모터부분에 해당하며
  - 즉, 프로펠러에 관한 rc control
  - 예를 들면, 4개 날개를 가진 drone이면 1~4까지 선이 꼽혀져있고 우리가 사용하는 교량 기체는 8개 날개를 가져서 1~8까지 선이 꼽혀져 있음
#### 🌟️ RC control in AUXOUT 🌟️
- 빨간 네모에서 노란색 줄이 쳐져있는 Auxout 부분이 external을 위한 rc control 파트
  - 실질적으로 1번이지만, rc기준 9번째이므로 rc 9~14번이라고 생각하면 되며 우리는 이를 건들면 됨  
- 9~14번까지 건드려서 pwm신호를 주면됨
  - 이를 이용하면 짐벌제어, 촬영 트리거, 카메라 다양한 기능, 열화상 카메라 on/off, 필터등... 다양한 기능을 사용할 수 있음
#### RC control using MAVROS
[MAVROS COMMAND](https://mavlink.io/en/messages/common.html)

![image](https://user-images.githubusercontent.com/108650199/216233883-0914e9f1-9103-4187-840a-abb53d2da7ab.png)

- mavros apm실행
- ```rosservice call /mavros/cmd/command "{broadcast: false, command: 0, confirmation: 0, param1: 0.0, param2: 0.0, param3: 0.0,
  param4: 0.0, param5: 0.0, param6: 0.0, param7: 0.0}" ``` 라는 service사용
- 여기서 183번을 사용하는데, param이 2개뿐임 Instance, PWM
  - 따라서, ```rosservice call /mavros/cmd/command "{broadcast: false, command: 183, confirmation: 0, param1: 9.0, param2: 1100.0, param3: 0.0,
  param4: 0.0, param5: 0.0, param6: 0.0, param7: 0.0}" ``` 이런식으로 사용함
  - command : 183번
  - param1 : Instance로 우리는 RC 9번에 연결했으므로 9
  - param2 : PWM로 MIN 1000, MAX 2000이지만 대부분 1100~1800 값을 사용
    - standby : 1100
    - 초점 : 1500 (middle값)
    - 찍기 :  
