![image](https://user-images.githubusercontent.com/108650199/225573103-67cb56d9-da46-4214-ac13-55683651cbe7.png)


## - 시나리오
- 현재 pose의 가는 방향의 값(imu의 linear acceleration)의 크기를 구해서 현재 pose에 값을 더하거나 빼줌
- 이후 이 new pose를 기준으로 radius search를 진행함
- radius search에서 point cloud size 10이상이면(값이 존재하면) stop

## in IMUPreintegration의 void imuOdometryHandler(const nav_msgs::Odometry::ConstPtr& odomMsg) 마지막
```
Eigen::Vector3f dir_vector;
dir_vector.x() = odomMsg->twist.twist.linear.x;
dir_vector.y() = odomMsg->twist.twist.linear.y;
dir_vector.z() = odomMsg->twist.twist.linear.z;

Eigen::Vector3f imu_dir_unit;

geometry_msgs::TwistStamped imu_dir;
imu_dir.header.stamp = odomMsg->header.stamp;
imu_dir.header.frame_id = baseFrame;
imu_dir.twist.linear.x = dir_vector.x()*3;
imu_dir.twist.linear.y = dir_vector.y()*3;
imu_dir.twist.linear.z = dir_vector.z()*3;


pubDirection.publish(imu_dir);
```

## in Mapoptimization
```
void dronedirHandler(const geometry_msgs::TwistStamped::ConstPtr& odomMsg)
{
    if (pubCollisionAvoidance.getNumSubscribers()!=0){
        Eigen::Vector3f dir_vector;
        dir_vector.x() = odomMsg->twist.linear.x;
        dir_vector.y() = odomMsg->twist.linear.y;
        dir_vector.z() = odomMsg->twist.linear.z;

        geometry_msgs::TwistStamped imu_dir;
        imu_dir.header.stamp = odomMsg->header.stamp;
        imu_dir.header.frame_id = baseFrame;
        imu_dir.twist.linear.x = dir_vector.x();
        imu_dir.twist.linear.y = dir_vector.y();
        imu_dir.twist.linear.z = dir_vector.z();

        PointTypePose thisPose6D = trans2PointTypePose(transformTobeMapped);
        pcl::PointXYZI point_xyz;
        pcl::PointXYZI drone_dir;
        point_xyz.x = -thisPose6D.x;
        point_xyz.y = -thisPose6D.y;
        point_xyz.z = thisPose6D.z;
        point_xyz.intensity = thisPose6D.intensity;        

        if(imu_dir.twist.linear.x < 0){
            drone_dir.x = point_xyz.x-imu_dir.twist.linear.x;
        }
        else if(imu_dir.twist.linear.x > 0){
            drone_dir.x = point_xyz.x-imu_dir.twist.linear.x;
        }

        if(imu_dir.twist.linear.y < 0){
            drone_dir.y = point_xyz.y-imu_dir.twist.linear.y;
        }
        else if(imu_dir.twist.linear.y > 0){
            drone_dir.y = point_xyz.y-imu_dir.twist.linear.y;
        }

        if(imu_dir.twist.linear.z > 0){
            drone_dir.z = point_xyz.z+imu_dir.twist.linear.z;
        }
        else if(imu_dir.twist.linear.z < 0){
            drone_dir.z = point_xyz.z-imu_dir.twist.linear.z;
        }

        // pubDirection.publish(imu_dir);

        pcl::PointCloud<PointType>::Ptr new_cloud(new pcl::PointCloud<PointType>());
        pcl::PointCloud<PointType>::Ptr new_cloud_filter(new pcl::PointCloud<PointType>());
        // for (const auto& point: *cloudKeyPoses6D){
        for (int i = 0; i < (int)cloudKeyPoses6D->size(); i++)
        {
            *new_cloud += *transformPointCloud(cornerCloudKeyFrames[i],  &cloudKeyPoses6D->points[i]);
            *new_cloud += *transformPointCloud(surfCloudKeyFrames[i],    &cloudKeyPoses6D->points[i]);
            downSizeFilterCorner.setInputCloud(new_cloud);
            downSizeFilterCorner.filter(*new_cloud);
        }

        pcl::ConditionAnd<PointType>::Ptr range_cond (new pcl::ConditionAnd<PointType>());
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("z", pcl::ComparisonOps::GT, drone_dir.z-20.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("z", pcl::ComparisonOps::LT, drone_dir.z+20.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("x", pcl::ComparisonOps::GT, -drone_dir.x-20.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("x", pcl::ComparisonOps::LT, -drone_dir.x+20.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("y", pcl::ComparisonOps::GT, -drone_dir.y-20.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("y", pcl::ComparisonOps::LT, -drone_dir.y+20.0)));

        pcl::ConditionalRemoval<PointType> condrem;

        condrem.setCondition (range_cond);
        condrem.setKeepOrganized(false);
        condrem.setInputCloud (new_cloud);
        condrem.filter (*new_cloud_filter);

        downSizeFilterCorner.setInputCloud(new_cloud_filter);
        downSizeFilterCorner.filter(*new_cloud_filter);

        // std::cout << "Original cloud number: " << new_cloud->points.size() << std::endl;
        // std::cout << "👉️ After condition cloud number: " << new_cloud_filter->points.size() << std::endl;

        pcl::KdTreeFLANN<PointType> kdtree;

        int radius = 5;
        std::vector<int> idx;
        std::vector<float> dis;
        pcl::PointXYZI drone_direction;

        kdtree.setInputCloud(new_cloud_filter);
        pcl::PointCloud<PointType>::Ptr drone_dir_filter(new pcl::PointCloud<PointType>());

        drone_direction.x = -drone_dir.x;
        drone_direction.y = -drone_dir.y;
        drone_direction.z = drone_dir.z;
        drone_direction.intensity = drone_dir.intensity;


        kdtree.radiusSearch(drone_direction, radius, idx, dis);
        for (const auto& idxes: idx){
            drone_dir_filter->points.push_back(new_cloud_filter->points[idxes]);
        }

        std::cout << "👉️👉️ Radius nearest neighbors: " << idx.size() << std::endl;

        publishCloud(pubCollisionAvoidance, drone_dir_filter, timeLaserInfoStamp, odometryFrame);

        std::cout<< drone_dir_filter->size() << std::endl;
        if(point_xyz.z>8 && drone_dir_filter->size() > 50){
            if(StopFlag==true){
                geometry_msgs::PoseStamped msg;
                // -로 넣어줘야됨 mavros토픽과 반대임..
                msg.pose.position.x = drone_direction.x;
                msg.pose.position.y = drone_direction.y;
                msg.pose.position.z = drone_direction.z;

                geometry_msgs::TwistStamped vel;
                vel.twist.linear.x = 0;
                vel.twist.linear.y = 0;
                vel.twist.linear.z = 0;
                vel.twist.angular.x = 0;
                vel.twist.angular.y = 0;
                vel.twist.angular.z = 0;

                std::cout << "🛑 STOP !!!!!!!!!! 🛑" << std::endl;

                pubVelStop.publish(vel);
                pubStop.publish(msg);

                StopFlag = false;
            }
        }
    }
}
```
