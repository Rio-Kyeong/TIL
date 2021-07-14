# XML Separation

```text
스프링에서는 스프링 설정파일을 이용해서 bean객체를 메모리 로딩을 하고 getBean을 이용해서 자바에서 쓰고 있기 때문에, 많은 코드가 하나의 xml파일에 담겨질 수 있다
하나의 xml파일에 너무 많은 내용이 담기다 보면 가독성과 효율의 문제가 발생한다.
```

## 스프링 설정 파일 분리방법


* 기존에는 applicationContext.xml 하나만 존재 
위 파일을, appCtx1.xml, appCtx2.xml, appCtx3.xml로 분리하면 효율적


* 각각의 기능별로 스프링 설정 파일(xml)을 분리
분리한 스프링 설정 파일들은 문자열 배열에 담아서 넣어주면 하나의 스프링 컨테이너로 사용가능
```java
String[] appCtxs = {"classpath:appCtx1.xml", "classpath:appCtx2.xml", "classpath:appCtx3.xml"};
​
GenericXmlApplicationContext ctx = 
            new GenericXmlApplicationContext(appCtxs);
```
* 스프링 설정 파일을 분리하는 다른 방법으로 xml파일에 import태그를 이용하는 방법이 있다
```java
// appCtxImport.xml
...
​
<import resource="classpath:appCtx2.xml"/>
<import resource="classpath:appCtx3.xml"/>
​
<bean id="studentDao" class="ems.member.dao.StudentDao" ></bean>
​
// studentDao  appCtx2.xml과 appCtx3.xml 설정 파일을 불러온다.
```

# 빈(Bean)의 범위
|**scope**|**explanation**
|:-----|:-----------|
|**singleton**|생성된 빈 객체는 한 개만 생성이 되고, getBean() 메소드로 호출될 때 하나의 객체가 같이 공유되므로 동일한 객체가 반환된다.|
|<span style="color:red">**prototype**</span>|싱글톤과 반대의 속성으로 getBean() 메소드를 이용할 때마다 새로운 객체가 계속 생성된다.|

## **프로토타입(bean에 scope속성을 이용하여 정의. 기본 값은 싱글톤)**

```xml
<bean id="dependencyBean" class="scope.ex.DependencyBean" scope="prototype">
    <constructor-arg ref="injectionBean" />
    <property name="injectionBean" ref="injectionBean" />
</bean>
```



