생성자(Constructor)
===
## 1. 생성자
- 해당 클래스의 객체가 인스턴스화될 때 자동으로 호출되는 특수한 종류의 멤버 함수
- 클래스를 정의한 후 클래스 객체를 생성하게 되면 메모리에 할당 됨
- 이때 클래스의 멤버 변수는 초기화되지 않은 상태이므로 사용할 수 없음
  - 클래스의 모든 멤버 변수가 모두 public인 경우 초기화를 직접 가능
  - but, private의 경우 외부 접근이 불가능한 상태라면 불가능
- 이럴 경우에는 객체를 사용하기 전에 무조건 해당 함수를 실행하고 객체를 사용해
- 👉️ 객체의 생성과 동시에 멤버 변수를 초기화해주는 멤버 함수인 생성자(constructor)사용 

## 2. 생성자 조건
- 생성자는 작성한 클래스명으로 선언할 경우 생성자가 됨 
- 💥️ 외부에서 클래스를 생성함과 동시에 동작해야 하므로 public영역에 생성 💥️
- 생성자는 리턴 타입이 없음
- 만약 생성자를 만들지 않는다면?
  - 생성자 없는 클래스란 있을 수 없음
  - 생성자가 없는 클래스에 대해서는, 컴파일러가 기본 생성자(default constructor)를 만들어 삽입하고, 자신이 삽입한 기본 생성자를 호출하도록 컴파일 

## 3. 생성자 종류
### - 기본 생성자
- 매개 변수를 갖지 않거나 모두 기본값이 설정된 매개 변수를 가지고 있는 생성자
- 클래스를 인스턴스화할 때 사용자가 초기값을 제공하지 않으면 기본 생성자가 호출
- Example
```
#include <iostream>

class Fraction
{
private:
    int m_numerator;   // 분자
    int m_denominator; // 분모

public:
    Fraction() // default constructor
    {
         m_numerator = 0;
         m_denominator = 1;
    }

    int getNumerator() { return m_numerator; }
    int getDenominator() { return m_denominator; }
    double getValue() { return static_cast<double>(m_numerator) / m_denominator; }
};

int main()
{
    Fraction frac; // Since no arguments, calls Fraction() default constructor
    std::cout << frac.getNumerator() << "/" << frac.getDenominator() << '\n';
         // Output: 0/1
    return 0;
}
```

### - 매개 변수가 있는 생성자를 사용한 초기화
- 기본 생성자는 클래스 멤버 변수의 기본값을 설정하는 데 유용하지만, 클래스 인스턴스 별 멤버 변수의 값을 특정한 값으로 초기화하고 싶은 경우 사용 
```
#include <cassert>

class Fraction
{
private:
    int m_numerator;
    int m_denominator;

public:
    Fraction() // default constructor
    {
         m_numerator = 0;
         m_denominator = 1;
    }

    // Constructor with two parameters, one parameter having a default value
    Fraction(int numerator, int denominator=1)
    {
        assert(denominator != 0);
        m_numerator = numerator;
        m_denominator = denominator;
    }

    int getNumerator() { return m_numerator; }
    int getDenominator() { return m_denominator; }
    double getValue() { return static_cast<double>(m_numerator) / m_denominator; }
};
```

## 4. 생성자 in sierra LIO-SAM
- in mapOptimization, 기본 생성자 호출해서 멤버 변수 초기화 
```
class mapOptimization : public ParamServer
{

public:

    // gtsam
    NonlinearFactorGraph gtSAMgraph;
    ...
    bool dataRecollecting = false;

    mapOptimization()
    {
    ...
    }
}    
    
```
