STATIC(정적 변수)
===

## 1. static 전역 변수 일 때
- 내부 링크가 있는 변수를 static 변수라고 함 
- static 변수는 변수가 정의된 소스 파일 내에서 어디서나 접근할 수 있지만, 소스 파일 외부에서는 참조할 수 없음 

## 2. static 지역 변수 일 때
- 원래, 지역 변수는 '자동 주기(auto duration)'를 가지며, 정의되는 시점에서 생성되고 초기화되며, 정의된 블록이 끝나는 지점에서 소멸
- static을 사용하면 '자동 주기(auto duration)'에서 '정적 주기(static duration)'로 바뀜
  -  이것을 정적 변수(static variable) 라고도 부름
- 💥️ 생성된 스코프(=범위)가 종료한 이후에도 해당 값을 유지하는 변수 💥️
- 💥️ 또한, 정적 변수는 한 번만 초기화되며 프로그램 수명 내내 지속 💥️

## 3. static 지역 변수 in sierra LIO-SAM
- in MapOptimization
```
void laserCloudInfoHandler(const lio_sam::cloud_infoConstPtr& msgIn)
{
    static bool roll_ready = false;
    static int init_count = 0;
    ...   
}
```
- static bool > false 저장된 값 계속 가지고 있음
  - defalt값이 true라도 초기화는 한 번만 되고 static bool에서 false로 바꿨으니 계속 false를 가짐 
