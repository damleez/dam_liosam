## Image Projection

### lasercloudIn
- lasercloudIn > raw cloud인 laserCloudMsg를 cloudQueue에 넣어서 Queue단위 만큼 잘림
- 그 cloudQueue의 맨 앞을 가져와서 currentCloudMsg라고 정의 그리고 cloudQueue버림
  - 즉, Queue만큼 단위로 잘라져서 laser cloud가 정의됨
- 그렇게 얻은 currentCloudMsg는 ROS msg이므로 pcl msg로 재정의 = laserCloudIn

### outlier removal filter (laserCloudIn)
- 1. 통계적 2. 거리(radius)기반

### ❓️Question❓️ 
- timeScanEnd = timeScanCur + laserCloudIn->points.back().time;
- 여기서 timeScanCur은 currentCloudMsg 헤더의 시간 (ROS msg)
- laserCloudIn은 cloudQueue의 front 부분들 (PCL) 
- 근데 laserCloudIn과 currentCloudMsg는 msg 형태만 바꾼거고 뭉구트려서 보면 같지 않나?
- laserCloudIn의 point의 back이란 어찌됐든 queue의 맨 앞 msg를 pcl로 바꾼건데 그 맨 앞 cloud point 단일의 point 에서도 front와 back시간이 존재하는건가? 
### 👉️ 해결
- 여기서 queue는 1scan이 아니라 여러개의 scan이 넣어져 있는 컨테이너
- 그래서 currentCloudMsg는 그 front의 의미가 여러개의 스캔 중 첫번째 스캔을 의미하고
- lasercloudin->point.back은 그 첫번째 스캔 중 마지막 point cloud를 의미

---

### Imuqueue
#### 참고사항
```
imuRPY2rosRPY : thisImuMsg > sensor_msgs/Imu.msg안에서 orientation뽑아서 getRPY
imuAngular2rosAngular : thisImuMsg > sensor_msgs/Imu.msg안에서 angular_velocity 뽑음
```
- scan사이에 imu queue가 있어야 하며, imu queue가 scan과 거의 맞물리는 순간(0.01초)이 front 그리고 다음 scan에서 맞물리고 더 나아가는 순간(scan+0.01초)내가 back
- for문 (i=0부터 queuesize만큼 i를 늘려감)에서 imuqueue[i] = thisImuMsg의 시간이 currentImuTime
  - imuTime <= scanCur (imu가 scan앞에 있지만 0.01초 내외)이면 thisImuMsg에서 ros msg 형태인 imuRoll, Pitch, Yaw값으로 정의하고 들고와 Init값으로 cloud.info에 저장
  - imuTime > scanEnd+0.01 (imu가 scan뒤에 있지만 0.01초 내외) 까지만 queue 유효
  - 또한, imuPointerCur을 정의하는데 == 0 (초기값)이면 RotX, Y, Z = 0이며 imutime = currentImutime으로 정의
    - 즉, 초기값에서는 IMU x,y,z=0 IMU r,p,y=r,p,y의 값으로 정의
    - 위와 같이 정의 후 ++imuPointerCur(전위 증감 연산자)로 넘어가는데 그러면 imuPointerCur이 0이 아니므로 밑의 식으로 넘어감
### imuAngular2rosAngular
- thisImuMsg를 ros msg인 angular x,y,z로 정의 
  - 위의 참고사항 참고 

### ❓️Question❓️ 
- queue가 한 바퀴 돌 때, queue의 처음은 init값 정의 (0,0,0,r,p,y) 후, 이후는 rot값을 angular이용해서 구하고 r,p,y는 그냥 구하는데 함수 끝나고 나서 (;) ```--imuPointerCur;```는 왜 --임? ++ 아닌가?

---

### Odomqueue
- odomQueue도 현재 scan(보다 조금 앞에 -0.01) 빠르면 pop front하다가, odom이 scan에 거의 가까이 왔을 때 pop
- odomQueue가 scan앞 0.01내외~scan 현재까지는 괜찮은데 현재 scan 넘어가면 return

### startOdomMsg
- 위에서 정의한 odomQueue[i] = startOdomMsg 이며, 진짜 현재 스캔 (-0.01~현재) 의 앞부분만 해당
  - 만약 timeScanCur과 같거나 넘어가면 break
  - 이러한 startOdomMsg에서 orientation을 뽑아 r,p,y를 얻고, position을 뽑아 x,y,z를 얻음 = TF

### endOdomMsg
- odomQueue.back이 현재 scan end 앞에 있으면 return
- endOdomMsg = odomQueue[i]라고 정의하고, timeScanEnd를 넘어가면 안됨 break
  - 즉, endOdomMsg에서 orientation을 뽑아 r,p,y를 얻고, position을 뽑아 x,y,z를 얻음 = TF
  - startTf와 endTF로 incre구해서, incre에서 x,y,z,r,p,y 얻음

### findRotation and findPosition
- findRotation : 선형보간(IMU와 관련), findPosition : curPos 정의(odom과 관련)
  - 그 때, 선배가 IMU 선형보간은 약간 의미가 없다고 했음 
    - 왜 ? 너무 자그만한 단위 (500Hz)로 쪼개져 있는데 그거 선형보간해봤자 간소한 차이
    - 💥️ 그러면 odom data를 선형보간 한다면? 💥️ 

### projectPointCloud()
- DeskewPoint : findRotation과 findPosition에서 얻은 현재 Rot, Pos값으로 incre구해서 newPoint정의
  - 이름 그대로 MAIN Deskewing 함수

---

## MapOptimization
- 
