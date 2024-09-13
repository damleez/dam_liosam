## void publishFrames()

> [reference](https://pcl.readthedocs.io/projects/tutorials/en/latest/remove_outliers.html)

![image](https://user-images.githubusercontent.com/108650199/220834680-fee364e9-b027-4fcd-abf1-0a7f18a6e450.png)


```
if (pubRecentKeyFrame.getNumSubscribers() != 0)
{   
    ros::Rate loop_rate(10);
    for (const auto& point: *cloudKeyPoses3D){
        pcl::PointCloud<PointType>::Ptr cloudOut(new pcl::PointCloud<PointType>());
        PointTypePose thisPose6D = trans2PointTypePose(transformTobeMapped);
        *cloudOut += *transformPointCloud(laserCloudCornerLastDS,  &thisPose6D);
        *cloudOut += *transformPointCloud(laserCloudSurfLastDS,    &thisPose6D);

        pcl::ConditionAnd<PointType>::Ptr range_cond (new pcl::ConditionAnd<PointType>());
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("z", pcl::ComparisonOps::GT, point.z-5.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("z", pcl::ComparisonOps::LT, point.z+5.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("x", pcl::ComparisonOps::GT, point.x-5.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("x", pcl::ComparisonOps::LT, point.x+5.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("y", pcl::ComparisonOps::GT, point.y-5.0)));
        range_cond->addComparison (pcl::FieldComparison<PointType>::ConstPtr (new pcl::FieldComparison<PointType> ("y", pcl::ComparisonOps::LT, point.y+5.0)));

        pcl::ConditionalRemoval<PointType> condrem;
        condrem.setCondition (range_cond);
        condrem.setKeepOrganized(false);
        condrem.setInputCloud (cloudOut);
        condrem.filter (*cloudOut);
        
        publishCloud(pubRecentKeyFrame, cloudOut, timeLaserInfoStamp, odometryFrame);

    }          
}    
```
- pubRecentKeyFrame이 local이며, 일단 네모 모양으로 잘랐음
  - 현재 pose기준 위, 아래, 양옆으로 5m씩
- 현재 input인 cloudOut에서 conditionAnd filter를 설정

### 과정
- 처음에 localization도 global걸 잘랐어서 global걸 잘랐음
    - 성공했지만, 생각해보니 글로벌은 의미가 없음 결국 목표는 local에서 위치 반경의 것들을 알고 그것들로 회피하는거니까
- local로 옮겨옴
    - SLAM킬 때, local이 cloud_registered로 되어있어서 관련된 pubRecentKeyFrame 쪽을 찾아 여기서 수정하기 시작 
- 처음에 cloudOut 가지고 해볼려고 했는데 안됐음
    - cout 시켜보니 6d pose고 라이다 반경에있는 x,y,z 좌표들을 들고오는 것 같았음
    - 그래서 찾아봐서 현재 pose가지고오는 pointType 정의된거 찾아서 (keypose3d) 그걸 기준으로 플마 오씩함 

### 문제
- ```ros::Rate loop_rate(10);``` 넣었는데 point pose 뽑아봤을 때 주르륵 뜸 ㅋ 안되는듯
