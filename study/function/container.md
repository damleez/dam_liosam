CONTAINER
===
## 1. container란?
- 컨테이너(Container)는 다른 객체들을(원소) 보관하는 하나의 커다란 보관소
- 한 컨테이너에는 한 가지 종류의 객체들만 보관
- 컨테이너는 자신이 보관하는 원소(element)들의 메모리를 관리하며, 각각의 원소에 접근할 수 있도록 멤버 함수를 제공
  - 원소에 접근하는 방법으로 크게 두 가지가 있음
    - 하나는 직접 함수를 호출해서 접근하는 것
    - 다른 하나는 반복자(iterator) 을 이용해서 접근하는 것

## 2. container의 종류

![image](https://user-images.githubusercontent.com/108650199/212865148-0908fc83-5f9b-4e33-b3cb-43164584f7da.png)

### 2.1 시퀀스 컨테이너 - vector
- vector 컨테이너 접근하기 위해 Iterator(반복자) 개념이 필요
- iterator는 컨테이너 원소에 접근할 수 있는 포인터와 같은 객체라고 볼 수 있음
- vector의 경우 begin(), end() 함수를 이용하여 반복자를 얻을 수 있음
  - begin() : vector의 첫번째 원소  
  - end() : vector의 마지막 원소 다음 위치 (
- ex) In sierra LIO-SAM, ```std::vector<int> pointSearchIndSaveMap;``` 정의 되어있을 때, 이 vector에 접근하기 위해 아래와 같이 iterator을 사용하여 접근
```
for (std::vector<int>::iterator iter = pointSearchIndSaveMap.begin(); iter != pointSearchIndSaveMap.end(); iter++) {
...
}
``` 

### 2.2 컨테이너 어댑터
- 이들은 다른 컨테이너 클래스들을 상속 받아서 다른 컨테이터 클래스의 객체에 특정한 인터페이스를 제공
- 이를 통해 원래 컨테이너의 기능을 제한하고, 어댑터가 제공하는 인터페이스를 사용
  - ex) stack 이 deque 에 작용한다면, deque에 stack 이 제공하는 top, pop, push 등의 인터페이스를 사용할 수 있게 되는 것

## 3. container in sierra-LIO-SAM
### 👉️ Map
- in MapOptimization, ```map<int, int> loopIndexContainer;```, ```map<int, pair<pcl::PointCloud<PointType>, pcl::PointCloud<PointType>>> laserCloudMapContainer;```

### 3.1 map container in 연관 컨테이너
- 노드 기반으로 이루어져있고 균형 이진 트리 구조(= 레드-블랙 트리)
  - 균형 이진 트리는 모든 노드의 왼쪽과 오른쪽 서브 트리의 높이가 1 이상 차이가 나지 않는 트리 

  ![image](https://user-images.githubusercontent.com/108650199/212868742-475f89dd-86bb-4253-8210-eb4417498255.png)

- map : key와 value가 쌍으로 저장
  - 🌟️ key는 고유한 값이므로 중복이 불가능 🌟️
  - first : key로 second : value로 저장
- map의 기본 구조는 map <key type, value type> 이름
- map에서 찾고자 하는 데이터가 있는지 확인하기 위해 ```iterator```사용 (begin, end())
  - ex) In sierra LIO-SAM, ```loopIndexContainer```는 컨테이너, ```for (auto it = loopIndexContainer.begin(); it != loopIndexContainer.end(); ++it)``` 
- map도 set과 마찬가지로 삽입이 되면서 자동으로 정렬
  - default는 less/오름차순 
- map container는 저장공간의 필요에 따라서 allocator 객체를 사용
- ex) map container

![image](https://user-images.githubusercontent.com/108650199/212869038-74013315-e909-4735-9671-a1323f68fa2a.png)

### 👉️ Pair with Vector
- in MapOptimization, ```vector<pair<int, int>> loopIndexQueue;``` 정의 후, 아래와 같이 first, second로 정렬
```
void addLoopFactor()
{
    if (loopIndexQueue.empty())
        return;

    for (int i = 0; i < (int)loopIndexQueue.size(); ++i)
    {
        int indexFrom = loopIndexQueue[i].first;
        int indexTo = loopIndexQueue[i].second;
        gtsam::Pose3 poseBetween = loopPoseQueue[i];
        gtsam::noiseModel::Diagonal::shared_ptr noiseBetween = loopNoiseQueue[i];
        gtSAMgraph.add(BetweenFactor<Pose3>(indexFrom, indexTo, poseBetween, noiseBetween));
    }

    loopIndexQueue.clear();
    loopPoseQueue.clear();
    loopNoiseQueue.clear();
    aLoopIsClosed = true;
}
```

### 3.2 pair container in 시퀀스 컨테이너 vector
- pair은 두개의 객체(first, second)를 하나로 묶어주는 역할을 하는 struct로 데이터의 쌍을 표현할 때 사용
- ex)
  - 벡터 선언 ```vector<pair<int, int>>v;```
  - 정렬 실행
  ```
  sort(v.begin(), v.end());              // 오름차순
  sort(v.begin(), v.end(), less<>());    // 오름차순
  sort(v.begin(), v.end(), greater<>()); // 내림차순
  ```
  - in sierra LIO-SAM, ```loopIndexQueue[i].first```라고하면 ```loopIndexQueue[i]```의 첫 번째 인자를 반환하며 vector형태이므로 ```[i]```임
