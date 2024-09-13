PCL LIBRARY
===
## 1. new, reset
### in 지역변수
```
pcl::PointCloud<PointType>::Ptr globalCornerCloud(new pcl::PointCloud<PointType>());
```

### in 전역변수
```
public:
pcl::PointCloud<PointType>::Ptr cloudKeyPoses3D;
```
```
void allocateMemory(){
cloudKeyPoses3D.reset(new pcl::PointCloud<PointType>());}
```
- 이렇게 전역변수에서는 바로 new못써줌
- 그 후, 생성자에서 초기화를 꼭 해줘야함
