Relationship with ext, rot
===

![image](https://user-images.githubusercontent.com/108650199/210705194-1bdb55aa-8175-4094-8d04-076131525d31.png)

## IMU converter function
```
    sensor_msgs::Imu imuConverter(const sensor_msgs::Imu& imu_in)
    {
        sensor_msgs::Imu imu_out = imu_in;
        // rotate acceleration
        Eigen::Vector3d acc(imu_in.linedar_acceleration.x, imu_in.linear_acceleration.y, imu_in.linear_acceleration.z);
        acc = extRot * acc;
        imu_out.linear_acceleration.x = acc.x();
        imu_out.linear_acceleration.y = acc.y();
        imu_out.linear_acceleration.z = acc.z();
        // rotate gyroscope
        Eigen::Vector3d gyr(imu_in.angular_velocity.x, imu_in.angular_velocity.y, imu_in.angular_velocity.z);
        gyr = extRot * gyr;
        imu_out.angular_velocity.x = gyr.x();
        imu_out.angular_velocity.y = gyr.y();
        imu_out.angular_velocity.z = gyr.z();
        // rotate roll pitch yaw
        Eigen::Quaterniond q_from(imu_in.orientation.w, imu_in.orientation.x, imu_in.orientation.y, imu_in.orientation.z);
        Eigen::Quaterniond q_final = q_from * extQRPY;
        imu_out.orientation.x = q_final.x();
        imu_out.orientation.y = q_final.y();
        imu_out.orientation.z = q_final.z();
        imu_out.orientation.w = q_final.w();

        if (sqrt(q_final.x()*q_final.x() + q_final.y()*q_final.y() + q_final.z()*q_final.z() + q_final.w()*q_final.w()) < 0.1)
        {
            ROS_ERROR("Invalid quaternion, please use a 9-axis IMU!");
            ros::shutdown();
        }

        return imu_out;
    }
};
```

## params.yaml
```
extrinsicRot: [-1, 0, 0,
                0, 1, 0,
               0, 0, -1]
extrinsicRPY: [0, -1, 0,
                1, 0, 0,
                0, 0, 1]
```

## 정리
### in feedback
- extrinsic RPY : IMU 제조사마다 다른 축에 대한 R, P, Y
- Rotation : Imu와 LiDAR사이의 회전값
### 나의 정리
- LiDAR는 오른손 법칙을 따라 x : forward, y : left, z : up 방향을 따름
- IMU는 x : forward, y : right, z : down 방향을 따름
#### extrinsicRot
- ROT이 imu와 LiDAR사이의 회전값이라면 x는 같으니 y와 z만 -를 붙이면 되는거 아닌가? 근데 왜 값이 다름..?
- 👉️ 내가 생각한 extrinsicRot
```
extrinsicRot: [ 1, 0, 0,
               0, -1, 0,
               0, 0, -1]
```
#### extrinsicRPY
- RPY가 IMU 축에 관한 R, P, Y이므로 LIDAR의 x,y,z 방향과 IMU의 x,y,z 방향을 비교해주면 됨
- 근데, 나 어디로 돌아야 + 방향인지 몰라서 모르겠음
- 뇌피셜로는 pitch와 yaw의 방향만 다름

#### 저자 피셜

> [reference](https://github.com/TixiaoShan/LIO-SAM/issues/6)

- extrinsic parameter가 두 개인 이유는 IMU의 inertial reading (acc 및 gyro) 프레임이 orientation frame과 다르기 때문
- IMU의 피치가 IMU-x를 중심으로 회전
- Lidar의 피치는 Lidar-y를 중심으로 회전지
- 따라서 extrinsicRPY를 사용하여 IMU 방향을 LiDAR 프레임에 대해 -90도 회전시켜 일관성을 유지

- 👉️ 내가 생각한 extrinsicRPY
```
extrinsicRPY :[ 1, 0, 0,
               0, -1, 0,
               0, 0, -1]
```

> ❓️ QUESTION ❓️

- extrinsicRPY에서 왜 1줄과 2줄을 바꾸면 안되는가
```
extrinsicRPY: [0, -1, 0,      →       extrinsicRPY: [1, 0, 0,
                1, 0, 0,                            0, -1, 0,
                0, 0, 1]                            0, 0, 1 ] 
```
- 애초에 imu x는 왜 pitch가 되는 것임..?


#### Reference
https://github.com/TixiaoShan/LIO-SAM/issues/344
- 근데 찾아도 안나옴
```
 // tixiao
      // Roboat IMU
      // IMU placement   ----     ROS
      // y ---  z (*)               x
      //        |                   |
      //        |                   |
      //        x             y ----z (x)
```

