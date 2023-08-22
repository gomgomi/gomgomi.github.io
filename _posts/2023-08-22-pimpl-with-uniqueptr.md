---
title: "unique_ptr로 Pimpl idiom 구현시 소멸자의 필요성"
categories:
    - C++
tags:
    - C++
    - pimpl
    - unique_tpr
    - forward declaration
---

Pimpl idiom을 사용하여 클래스를 아래와 같은 형태로 작성을 했다.
``` c++
// header
class Controller
{
public:
	Controller();
private:
	class ControllerImpl;
	std::unique_ptr<ControllerImpl> d;
};

// cpp
#include "Controller.h"

class ControllerImpl
{
	...
};

Controller::Controller() : d(new ControllerImpl) {
	...
}
```

그리고 컴파일을 했는데.. 어?
아래와 같은 오류가 발생했다.
```
use of undefined type 'ControllerImpl'
can't delete an incomplete type
```

# 문제 파악
구글링 해본 결과, 오류 발생 이유가 아래 두 가지인 것을 알아냈다.
1. 컴파일러가 자동생성하는 소멸자를 사용
2. ControllerImpl을 forward declaration으로 사용

우선, **컴파일러가 자동생성하는 소멸자**와 **Forward declaration**이 무엇인지 간단히 살펴보자.

## 컴파일러 자동생성 소멸자
클래스 생성시 작성하지 않아도 컴파일러가 자동으로 생성하는 특수 멤버함수가 있다. 생성자, 소멸자, 복사 생성자, 복사 대입 연산자가 이에 해당한다.  
이 특수 멤버함수를 컴파일러가 생성할 때는 public, inline 함수로 생성된다.

위 오류가 발생했을 때 소멸자를 작성하지 않아 컴파일러가 생성하는 것이 사용되었다.  
소멸자에서 특별히 해야하는 동작이 없어서 소멸자를 따로 작성하지 않았는데, 이게 원인 중 하나가 될 것이라곤 생각도 못했다.

## Forward declaration
위 Pimpl로 작성한 예제 코드의 header에 `class ControllerImpl`라고 작성한 부분이있다.  
이처럼, 실제 `ControllerImpl`을 정의하기 전에 미리 선언하여 컴파일러가 header만 보고 식별할 수 있도록 하는 방법이다. 

## 그래서 왜?
컴파일러 입장에서 한 번 생각해보자.

컴파일러는 소멸자가 없으니 직접 inline으로 소멸자를 생성한다.  
그리고 unique_ptr로 선언된 `ControllerImpl`을 소멸자에서 삭제해야되니 type을 확인한다.  
그런데 `ControllerImpl`은 forward declaration으로 되어있네? 그럼 컴파일러는 이것을 Incomplete type으로 인식한다.  
알 수 없는 type이므로 컴파일러는 '난 이거 삭제 못해! 그러니 컴파일도 못해!'가 되어버린다.

# 해결방안
그렇다면 어떻게 해결해야하나?

매우 간단하다. inline으로 자동 생성되지 않도록 코드에 소멸자를 만들어 주면 된다!  
다만, `ControllerImpl` 정의된 후에 소멸자를 작성해주어야 한다. 

문제를 수정하면 아래와 같은 코드가 된다.
``` c++
// header
class Controller
{
public:
	Controller();
    ~Controller();
private:
	class ControllerImpl;
	std::unique_ptr<ControllerImpl> d;
};

// cpp
#include "Controller.h"

class ControllerImpl
{
	...
};

Controller::Controller() : d(new ControllerImpl) {
	...
}

Controller::~Controller() = default;
```

---
Ref.
* https://www.fluentcpp.com/2017/09/22/make-pimpl-using-unique_ptr/