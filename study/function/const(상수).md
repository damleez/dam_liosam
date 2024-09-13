CONST
===

## 0. 용어 정리
- 멤버 변수 = 전역 변수 : 선언위치가 '클래스 영역'
- 지역 변수 : 선언위치가 '메소드나 생성자 내부'

## 1. const
- 그 대상을 변경하지 않는 "상수"를 의미

## 2. const의 사용
### 2.1 const 변수
#### - const 지역 변수

![image](https://user-images.githubusercontent.com/108650199/213092831-56dbf061-6203-4b4f-aa44-c93dd127f3d4.png)

- 위의 두 선언 방식은 의미가 같은데 보통 const 위치가 맨 앞인 첫 번째 형식을 많이 사용
- 위와 같이 선언하면 ```num```은 변할 수 없는 상수가 되는 것

#### - const 멤버 변수

![image](https://user-images.githubusercontent.com/108650199/213093370-620911b2-fcb7-4c44-90c3-8d98b70d7d8a.png)

- const 변수는 반드시 선언 시 초기화를 해야 하며 초기화가 되지 않으면 컴파일 에러가 발생
- 그래서 class의 멤버 변수를 const로 선언 시에는 반드시 초기화 리스트(Initialize List)를 사용
- Foo의 num은 생성자에서 초기화 리스트로 1로 초기화 하고 있는데, 이는 const int num = 1;과 동일한 의미


#### - const 포인터 변수
- 포인터와 const를 사용할 때는 두가지 경우가 있음
##### 첫 번째, const 위치가 맨 앞에 있으면서 포인터 변수가 가리키는 값에 대하여 상수화를 시키는 경우

![image](https://user-images.githubusercontent.com/108650199/213093855-382ecb36-c2e6-442c-8973-6f495f36f0e5.png)

- ```*ptr = 2;```는 ptr이 const 변수이기 때문에 컴파일 에러가 발생 하지만 num = 2;는 num이 non-const 변수이기 때문에 정상
- 즉, 포인터 변수가 가리키는 num 자체가 상수화가 되는 것이 아님

##### 두 번째, const 위치가 type과 변수 이름 사이에 있으면서 포인터 변수 자체를 상수화 시키는 경우

![image](https://user-images.githubusercontent.com/108650199/213094092-3a120ab5-31ec-4a6a-ad80-95e0a77013e0.png)

- 포인터 변수란 대상의 주소 값을 저장하는 변수
- 즉, 위의 ptr은 자기 자신을 상수화 시키는 것이기 때문에 num2의 주소값으로 변경하려고 하면 컴파일 에러가 발생

### 2.2 const 멤버 함수
#### - 함수 선언에 사용되는 const

![image](https://user-images.githubusercontent.com/108650199/213094435-534d29f9-4b97-422b-b65d-9b6fa414c706.png)

- const 멤버 함수는 이름 그대로 class의 멤버 함수만 const로 상수화를 시킬 수 있고 지역 함수는 const 함수로 선언이 불가능
  - ‼️ 여기서, class 내부에 있으면 다 멤버함수임 ‼️
- const의 위치는 함수 선언문 맨 뒤이고 되는데 이 의미는 해당 멤버 함수 내에서는 모든 멤버 변수를 상수화 시킨다는 의미
- 따라서 위와 같이 멤버 변수인 num은 변경할 수 없고 지역 변수인 a는 변경할 수 있는 것

#### - 매개변수에 붙어있는 const
- ex ) ```void Func(const int& nNum)``` 
#### In-parameter vs out-parameter
##### In-parameter
```
void Output(int nNum)
{
  cout << nNum << endl;
}
int main(void)
{
  int nNum = 10;
  Output(nNum);
  return 0;
}
```
- 함수에 nNum이란 data를 전달하는 것 

##### Out-parameter
```
void Output(int* nNum)
{
  *nNum = 5;
}
int main(void)
{
  int nNum = 10;
  Output(&nNum);
  cout << nNum << endl;
  return 0;
}
```
- 함수에 의해 nNum이라는 data의 값이 바뀌는 것
##### 정리하면,
- In-paramter : 값, 주소, 레퍼런스 전달
- out-parameter : 주소, 레퍼런스 전달
- 💥️ 즉, in-parameter로 포인터 혹은 레퍼런스를 사용하고 싶은데, 값 변경의 위험이 느껴지면 매개변수 앞에 const를 붙여 상수화 💥️
##### 👉️ const in-parameter
```
void Output(const int& nNum)
{
  cout << nNum << endl;
}
int main(void)
{
  int nNum = 10;
  Output(nNum);
  cout << nNum << endl;
  return 0;
}
```
- 이렇게 하면 nNum의 값 변경을 시도하려고 할 때, 에러가 발생되어 값 변경 불가능
### 💥️💥️ 즉 결론 ! in-parameter로 쓰이는 pointer와 reference는 꼭 const를 붙일 것 ! 💥️💥️ 

## 3. const in sierra LIO-SAM
- class안의 멤버함수의 매개변수인 pointer에 const를 붙여서 값 지킴 
```
void laserCloudInfoHandler(const lio_sam::cloud_infoConstPtr& msgIn)
```
