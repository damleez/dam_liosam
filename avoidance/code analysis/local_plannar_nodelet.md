## 1. 소멸자
- thread 관련된 친구들 정리
  - buffer, transformed cloud, camera
  - 이후 worker joinable, join을 통해 활성 thread인지 확인

## 2. 초기화 onInit()
- 함수 3개 정의 2.1~2.3 InitializeNodelet, init_streamrate, startNode
- worker 정의 
  - 소멸자에서 사용
- server 정의
  - dynamic reconfigure 로 정의 > f
    - 📚 server가 동작하는 동안 실시간 node parameter update로 중간에 parameter gain 외부에서 즉시 수정 가능
  - f를 boost::bind로 바인딩화 > dynamicReconfigureCallback
    - 📚 boost::bind : 이용하면 함수 포인터에 비해 유연성이 상당히 좋아진다는 것

### 2.1 InitializeNodelet()
- reset함수를 사용하여 다른 cpp파일의 클래스?함수?(LocalPlanner, WaypointGenerator, AvoidanceNode) 객체에 대한 소유권 획득
  - 다른곳에서 reset하면 다시 새롭게 진짜 reset해서 사용하는 것임 즉, pointer 교체
- RadiusOutlierRemoval filter 사용
  - set radius, set min neighbor 정의
- tf listener 정의, sub, pub, service 정의 
- local_planar cpp의 클래스의 setDefaultParameters 및 applyGoal함수에 접근
  - 📚 reset이 스마트 포인터 이므로 ```포인터이름->멤버함수``` 요로케 접근
- setSystemStatus > System is booting up 
  - MAV_STATE::MAV_STATE_BOOT  
- hover, planner is healthy, armed, start time 정의

### 2.2 init_streamrate()
- mavros_msgs::StreamRate 정의
  - 그 밑 파라미터?인 stream id, message rate, on off정의 (0,3,1 or 10,6,1)

### 2.3 startNode()
- time과 관련된 함수 cmdLoopCallback 정의
  - timer, cmdloop spinner reset > start > init 정의

## 3. readParams()
- launch file에 정의되어있는 parameter 들고와서 읽음 
  - goal x,y,z param, filter ...
  - 🌟️ goal x param > goal_d.x() 라고 정의한 이후 이 x,y,z들을 모아서 goal_position_ 정의 🌟️
- camera topic을 pointcloud_topics라고 정의하며 이 변수는 3.1 initializeCameraSubscribers에 정의 됨
- new_goal_ = true라고 init

### 3.1 initializeCameraSubscribers()
- cameras는 h에 저장된 vector형으로 topic 및 point cloud XYZ의 unTF, TF 등이 저장되어 있음
- camera_topics.size 크기만큼 for문 
  - 🌟️ point cloud sub 정의 in pointCloudCallback 함수 🌟️
  - 🌟️ transform thread 정의 in pointCloudTransformThread 함수 🌟️

### 3.1.1 pointCloudCallback()
#### timeSinceLast 최신시간 계산
- auto로 정의하며 ```auto 변수 = [&]() -> 함수``` 를 해석하자면
  - auto함수 함수내부에서 외부의 지역변수 참조시 ```[&]```를 사용하며, 파라미터가 존재하지 않아 ()안에 파라미터들이 정의 X (매개변수 존재시 ex (int a))
  - 무튼, 이 함수 안에서 lastCloudReceived를 정의
    - lastCloudReceived : pcl point cloud인 ```cameras_[index].untransformed_cloud_.header.stamp```를 ROS형으로 바꿈
  - 이후 ```return msg->header.stamp - lastCloudReceived;```을 통해 현재 pointcloud msg의 stamp(time)과 카메라에서 받은 stamp(time)과의 차이를 계산하여 return시킴
####
 
### 3.1.2 pointCloudTransformThread()
