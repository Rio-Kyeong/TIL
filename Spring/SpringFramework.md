# SpringFramework
## Framework
```text
개발자들이 개발을 하기위한 어떠한 업무를 추상적으로 정의 해 놓은 틀
틀 안에서 내가 필요한 기능만 구현하여 개발할 수 있다(개발이 편리하다)
```
## 스프링 프레임워크의 특징
POJO
```text
POJO는 gettet/setter를 가진 단순 자바 오브젝트로 정의를 하고 있다. 이러한 단순 오브젝트는 의존성이 없고 추후 테스트 및 유지보수가 편리한 유연성의 장점을 가집니다. 이러한 장점들로 인해 객체지향적인 다양한 설계와 구현이 가능해지고 POJO의 기반의 Framework가 조명을 받고 있다.
```
AOP
```text
AOP(Aspect Oriented Programming)란 말 그대로 관점 지향 프로그래밍이다.

OOP(Object Oriented Programming)는 관심사가 같은 데이터를 한곳에 모아 분리하고 낮은 결합도를 갖게하여 독립적이고 유연한 모듈로 캡슐화를 하는 것을 일컫는데, 이러한 과정 중 중복된 코드들이 많아지고 가독성, 확장성, 유지보수성을 떨어 뜨린다. 이러한 문제를 보완하기 위해 나온 것이 AOP이다.

AOP에서는 핵심기능과 공통기능을 분리시켜 핵심 로직에 영향을 끼치지 않게 공통기능을 끼워 넣는 개발 형태 이며 이렇게 개발함에 따라 무분별하게 중복되는 코드를 한 곳에 모아 중복 되는 코드를 제거 할 수 있어지고 공통기능을 한 곳에 보관함으로써 공통 기능 하나의 수정으로 모든 핵심기능들의 공통기능을 수정 할 수 있어 효율적인 유지보수가 가능하며 재활용성이 극대화된다.
```
MVC(model2)
```
Model
Model은 데이터처리를 담당하는 부분으로 Serivce영역과 DAO영역으로 나눠진다.
Model에서는 View와 Controller 어떠한 정보도 가지고 있어서는 안된다.

View
View는 사용자 Interface를 담당하며 사용자에게 보여지는 부분이다.
Model이 가지고 있는 정보를 저장해서는 안되며 Model, Controller에 구성 요소를 알아서는 안된다.

Controller
Controller에서는 View에 받은 요청을 가공하여 Model(Service 영역)에 이를 전달한다.
또한 Model로 부터 받은 결과를 View로 넘겨주는 역할을 합니다.
Controller에서는 모든 요청 에러와 모델 에러를 처리하며 Model과 View의 정보를 알고 있어야한다.
```


## 스프링 프레임워크의 구조(모듈)
|**스프링 모듈**|**기능**|
|------|-----------------|
|spring-core|스프링의 핵심인 DI(Dependency Injection)와 IOC(Inversion of Control)를 제공|
|spring-context|Spring Framework의 context 정보들을 제공하는 설정 파일 (Spring Context에는 JNDI, EJB, Validation, Scheduiling, Internaliztaion 등 엔터프라이즈 서비스들을 포함)|
|spring-aop|AOP구현 기능 제공 (관점 지향 프로그래밍 - 핵심기능과 공통기능 분리)|
|spring-jdbc|JDBC를 보다 단순화하여 기능을 제공|
|spring-orm|Object relational mapping의 약자로 간단하게 객체와의 관계 설정을 하는 것 (Ibatis, Hibernate, JDO 등 인기있는 객체 관계형 도구(OR도구)를 사용 할 수 있도록 지원)
|spring-web|Web기반의 응용프로그램에 대한 Context를 제공|
|spring-mvc|Model2 구조로 Apllication을 만들 수 있도록 지원 (스프링MVC 구현 기능 제공)|

```text
스프링 프레임워크에서 제공하고 있는 모듈을 사용하려면, 모듈에 대한 의존설정을 개발 프로젝트에 XML 파일등을 이용해서 개발자가 직접 하면 된다.
```

## Inversion of Control(Ioc)
**IOC는 DI(의존성 주입)와 DL(의존성 검색)의 의해 구현됩니다.**
```text
IoC는 제어의 역전이라고 말하며, 간단히 말해 "제어의 흐름을 바꾼다"라고 한다.

메소드나 객체의 호출작업을 개발자가 결정하는 것이 아니라, 외부에서 결정되는 것을 의미한다
(제어의 흐름을 사용자가 아닌 스프링이 처리한다.)

객체의 의존성을 역전시켜 객체 간의 결합도를 줄이고(약결합) 유연한 코드를 작성할 수 있게 하여 가독성 및 코드 중복, 유지 보수를 편하게 할 수 있게 한다.
```

## Spring Container
```text
스프링에서 객체를 생성하고 관리하는 컨테이너(Container)로, 컨테이너를 통해 생성된 객체를 빈(Bean)이라고 부른다.
```
![Container](https://github.com/RyuKyeongWoo/TIL/blob/main/Spring/img/Spring%20Container.PNG)
