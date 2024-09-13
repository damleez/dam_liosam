## 1. 이동하는 방향 표현
```
for (int i = 0; i < (int)cloudKeyPoses6D->size(); i++){
    geometry_msgs::TwistStamped dir_vec;
    // Eigen 안됨 888
    Eigen::Vector3f cur_vec;
    cur_vec.x() = cloudKeyPoses6D->points[i].x;
    cur_vec.y() = cloudKeyPoses6D->points[i].y;
    cur_vec.z() = cloudKeyPoses6D->points[i].z;

    Eigen::Vector3f pre_vec;
    pre_vec.x() = cloudKeyPoses6D->points[i-5].x;
    pre_vec.y() = cloudKeyPoses6D->points[i-5].y;
    pre_vec.z() = cloudKeyPoses6D->points[i-5].z;

    Eigen::Vector3f final_vec;
    final_vec.x() = cur_vec.x()-pre_vec.x();
    final_vec.y() = cur_vec.y()-pre_vec.y();
    final_vec.z() = cur_vec.z()-pre_vec.z();
    
    dir_vec.header.frame_id = baseFrame;
    dir_vec.twist.linear.x = final_vec.x();
    dir_vec.twist.linear.y = final_vec.y();
    dir_vec.twist.linear.z = final_vec.z();

    pubDirection.publish(dir_vec);

}
```
- 현재 위치 - 이전 위치 => vector에 넣어서 pub

