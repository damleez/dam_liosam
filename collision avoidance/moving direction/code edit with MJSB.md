## 1. imuPreintegration
- in imuOdometryHandler 에서 rotationMatrix 추가
  - 왜? 최종적으로 linear acceleration을 구했을 때, 좌표계의 방향이 현재 우리가 사용하고 있는 좌표계랑 다르기 때문
  - slam은 lidar pose를 구하는 것 즉 lidar pose가 기준이 됨
  - 근데 linear accelration같은 경우 만들어진 map을 기준이 되기 때문에 yaw 180도 만큼 차이가 남 
```
Eigen::Matrix3f rotationMatrix;
rotationMatrix = Eigen::AngleAxisf(rotationMTO.x(), Eigen::Vector3f::UnitX())
               * Eigen::AngleAxisf(rotationMTO.y(), Eigen::Vector3f::UnitY())
               * Eigen::AngleAxisf(rotationMTO.z(), Eigen::Vector3f::UnitZ());
Eigen::Vector3f dir_vector(odomMsg->twist.twist.linear.x, odomMsg->twist.twist.linear.y, odomMsg->twist.twist.linear.z);
dir_vector = rotationMatrix * dir_vector;
```
#### - 특이점
- Eigen matrix정의시, 먼저 정의하고 함수 좌르륵 해야되더라,,? 정의와 동시에 선언(?)하면 error 발생
#### - 결과
- 기호 문제 발생하지 않음
