COORDINATE SYSTEM GPS vs NON-GPS
===

## 1. coordinate system

![image](https://user-images.githubusercontent.com/108650199/215028529-b3f80346-a523-4825-bc77-213c52d05e80.png)

### example
#### - GPS O
- ```(-50,30)```으로 ```rostopic pub /move_base_simple/goalgoal geometry_msgs/PoseStamped```을 찍었을 때, enu frame이므로 10시 방향으로 날라감
#### - NON-GPS
- GPS가 없으므로 SLAM좌표계를 따름
  - x : north, y : left
- 똑같이 10시 방향으로 날라가려면 ```(50,30)```으로 날리면 됨
