CLASS, OBJECT, INSTANCE
===
## 1. Class
- 사용자 정의 데이터 유형으로 데이터 멤버 및 멤버 함수가 포함
- 해당 클래스의 객체(Object 또는 Instance)를 생성하여 접근(Access)하고, 사용할 수 있음
  - ex) 자동차 class 선언과 자동차 class의 멤버변수들
  ```
  class Car
  {
  public:
    std::string brand;
    std::string color;
    int maxPerson;
  };
  ```
- 클래스가 정의 될 때 메모리에 할당되지 않으며 객체가 생성될 때 메모리가 할당

## 2. 객체
- 객체 생성 : 클래스 이름 + 객체 이름
- 객체의 멤버 접근 : 멤버 연산자 '.'
  - ex )

  ![image](https://user-images.githubusercontent.com/108650199/212881954-1a2df127-5a82-4bb6-96cb-51be05f151ca.png)

## 3. 정리

![image](https://user-images.githubusercontent.com/108650199/212886223-255eb383-802a-4424-b5bd-09807d0af148.png)

- 좀 더 정확히는 메모리상에 구현된 객체를 인스턴스라고 함
  - 객체가 인스턴스를 포함
  - 위와 같은 상태에서는 tmpClass가 객체이고, (10)이 인스턴스
- 💥️ 즉, 객체는 클래스 타입으로 선언되었을 때를 의미하고, 인스턴스는 인자를 받아 메모리에 할당되어 실제 사용 가능한 상태 💥️

## 4. class in sierra LIO-SAM
```
class mapOptimization : public ParamServer
{
...
bool aLoopIsClosed = false;      // 멤버 함수와 멤버 함수의 원형
lio_sam::cloud_info cloudInfo;   // 멤버 변수
...
}
```

