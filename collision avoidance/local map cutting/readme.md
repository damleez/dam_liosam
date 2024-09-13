## 결과

![rviz_screenshot_2023_03_02-16_47_57](https://user-images.githubusercontent.com/108650199/222365241-5149f84f-5540-478e-a8bb-c9c5e5ec8d7f.png)

![image](https://user-images.githubusercontent.com/108650199/222365359-3f18e05c-b922-474a-8798-224e7cdcbbf2.png)

### 1. cloudOut 저장
```
if (pubRecentKeyFrame.getNumSubscribers() != 0)
{   
    ros::Rate loop_rate(10);

    pcl::PointCloud<PointType>::Ptr cloudOut(new pcl::PointCloud<PointType>());
    pcl::PointCloud<PointType>::Ptr cloudOut_filter(new pcl::PointCloud<PointType>());
    // for (const auto& point: *cloudKeyPoses6D){
    for (int i = 0; i < (int)cloudKeyPoses6D->size(); i++)
    {
        *cloudOut += *transformPointCloud(cornerCloudKeyFrames[i],  &cloudKeyPoses6D->points[i]);
        *cloudOut += *transformPointCloud(surfCloudKeyFrames[i],    &cloudKeyPoses6D->points[i]);
    }
```


### 2. conditionAnd로 네모로 먼저 자름 
```
    PointTypePose thisPose6D = trans2PointTypePose(transformTobeMapped);
    pcl::ConditionAnd<PointType>::Ptr range_cond (new pcl::ConditionAnd<PointType>());
    range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("z", pcl::ComparisonOps::GT, thisPose6D.z-15.0)));
    range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("z", pcl::ComparisonOps::LT, thisPose6D.z+15.0)));
    range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("x", pcl::ComparisonOps::GT, thisPose6D.x-15.0)));
    range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("x", pcl::ComparisonOps::LT, thisPose6D.x+15.0)));
    range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("y", pcl::ComparisonOps::GT, thisPose6D.y-15.0)));
    range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("y", pcl::ComparisonOps::LT, thisPose6D.y+15.0)));

    pcl::ConditionalRemoval<PointType> condrem;

    condrem.setCondition (range_cond);
    condrem.setKeepOrganized(false);
    condrem.setInputCloud (cloudOut);
    condrem.filter (*cloudOut_filter);

    downSizeFilterCorner.setInputCloud(cloudOut_filter);
    downSizeFilterCorner.filter(*cloudOut_filter);

    std::cout << "After condition cloud number: " << cloudOut_filter->points.size() << std::endl;
```

### 3. kdtree radius search 실행

> [reference](https://limhyungtae.github.io/2021-09-12-ROS-Point-Cloud-Library-(PCL)-8.-KdTree%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Radius-Search/)

```
    pcl::KdTreeFLANN<PointType> kdtree;

    int radius = 5;
    std::vector<int> idx;
    std::vector<float> dis;
    pcl::PointXYZI point_xyz;

    kdtree.setInputCloud(cloudOut_filter);
    pcl::PointCloud<PointType>::Ptr cloud_filter(new pcl::PointCloud<PointType>());

    point_xyz.x = thisPose6D.x;
    point_xyz.y = thisPose6D.y;
    point_xyz.z = thisPose6D.z;
    point_xyz.intensity = thisPose6D.intensity;

    kdtree.radiusSearch(point_xyz, radius, idx, dis);
    for (const auto& idxes: idx){
        cloud_filter->points.push_back(cloudOut_filter->points[idxes]);
    }

    std::cout << "👉️ Radius nearest neighbors: " << idx.size() << std::endl;

    publishCloud(pubRecentKeyFrame, cloud_filter, timeLaserInfoStamp, odometryFrame);
}    
```
