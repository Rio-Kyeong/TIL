# Implementing Basic Authentication with Spring Security
<pre>
Spring Security는 Spring 기반의 <b>애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크</b>이다.
Spring Security는 <b>'인증'과 '권한'에 대한 부분을 Filter 흐름에 따라 처리</b>하고 있다.

Filter는 Dispatcher Servlet으로 가기 전에 적용되므로 가장 먼저 URL 요청을 받지만,
Interceptor는 Dispatcher와 Controller사이에 위치한다는 점에서 적용 시기의 차이가 있다.

Reference : <a href="https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kimnx9006&logNo=220634017538">내용1</a> <a href="https://postitforhooney.tistory.com/entry/SpringSecurity-%EC%B4%88%EB%B3%B4%EC%9E%90%EA%B0%80-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-Spring-Security-%ED%8D%BC%EC%98%B4">내용2</a> <a href="https://catsbi.oopy.io/c0a4f395-24b2-44e5-8eeb-275d19e2a536">내용3</a> <a href="https://gs.saro.me/dev?page=20&tn=480">내용4</a>
</pre>
## Security
### pom.xml
```xml
<!-- 서비스가 기동되면 스프링 시큐리티의 초기화 작업 및 보안 설정이 이뤄진다 -->
<!-- 별도의 설정이나 구현을 하지 않아도 기본적인 웹 보안 기능이 현재 시스템에 연동된다 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
### Configuration Class
```java
package com.example.restfulwebservice.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration //SpringBoot 가 기동을 하면서 메모리에 설정정보를 같이 로딩을 한다.
@EnableWebSecurity //웹 보안 활성화
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        //모든 사용자가 접근할 수 있는 URL 패턴을 지정한다("/h2-console 로 시작할 경우 사용자는 해당 요청에 접근할 수 있다)
        http.authorizeRequests().antMatchers("/h2-console/**").permitAll();
        http.csrf().disable(); //크로사이드 스크립트 관련된 것들 사용하지 않는다.
        http.headers().frameOptions().disable(); //헤더 값의 프레임에 관련된 속성 값을 사용하지 않는다.
    }
    
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth)throws Exception{
        auth.inMemoryAuthentication()
                .withUser("Kenneth") //Username(id)
                .password("{noop}test1234") //{noop} : 인코딩 없이 처리
                .roles("USER"); //로그인이 완료되면 USER 권한을 부여
    }
}
```
### Postman
<pre>
<img src="https://github.com/RyuKyeongWoo/TIL/blob/main/SpringBoot/img/security.PNG"/>
</pre>
