## 1. 전체코드
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
```

### 1.1. 문제점
- 네모낳게 자른 후, cloudOut_filter의 size가 0이되면 뻑이나는 현상 발생
- radius이후 idx의 size를 cout해보니 잘라진 것을 발견함
  - 즉, 위에서 자른 cldouOut_filter가 동그랗게 잘라져서 나오는게 아니라 네모낳게 잘린거가 바로 걍 내려오는것임 ㅇㅇ

### 1.2. 해결
- 네모를 좀 더 크게 잘라서 인풋클라우드가 뻑날일이 없게끔 설정
- 새로 pcl ptr받아가지고 거기에 cloudOut_filter의 idx에 해당하는 points 저장한 후, 이 새로운 pcl ptr을 pub해줌 !

## 2. 핵심코드
```
kdtree.radiusSearch(point_xyz, radius, idx, dis);
for (const auto& idxes: idx){
    cloud_filter->points.push_back(cloudOut_filter->points[idxes]);
}
```
- 이후 cloud_filter를 publish해주면 동그랗게 나옴!


## 3. 결과

![rviz_screenshot_2023_03_02-16_47_57](https://user-images.githubusercontent.com/108650199/222367469-8183e233-e93c-4f86-87e2-7f63a4a07025.png)
