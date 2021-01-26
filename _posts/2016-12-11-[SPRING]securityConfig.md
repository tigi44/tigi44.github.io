---
title: "[SPRING] JAVA SECURITY CONFIG"
excerpt: "Spring framework - java security config"
description: "Spring framework - java security config"
modified: 2017-01-08
categories: "Spring"
tags: [spring framwork, java config, security config, spring4, java1.8, Server]

toc: true
toc_sticky: true

header:
  teaser: /assets/images/OG-Spring.png
---

## SECURITY CONFIG
JAVA를 이용한 Spring Security Config 설정

---

## GIT SOURCE
* [springStudy - Security Config commit](https://github.com/onlytigi/springStudy/commit/ad38c24b83983fa6d692af27dd11bb6e548f5e8a)

---

## VERSION
- JAVA Version : 1.8
- Spring Framework Version : 4.x.x

---

## MAVEN 설정 (pom.xml)
Spring Framework security 관련 maven 설정 필요

```xml
<!-- Security -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-web</artifactId>
	<version>4.0.0.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-config</artifactId>
	<version>4.0.0.RELEASE</version>
</dependency>
```

---

## WebConfig.java (web.xml 대체)
서블릿 필터 등록시 필터 내부에서 Spring Bean을 주입 받기 위해 DelegatingFilterProxy 등록 필요

```java
// web.xml 대체용 WebConfig.java내 DelegatingFilterProxy 등록 코드

public class WebConfig implements WebApplicationInitializer {

	...

//	<!-- 기존 web.xml에서 DelegatingFilterProxy 등록  -->
//	<filter>
//		<filter-name>springSecurityFilterChain</filter-name>
//		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
//	</filter>
//	<filter-mapping>
//		<filter-name>springSecurityFilterChain</filter-name>
//		<url-pattern>/*</url-pattern>
//	</filter-mapping>
	FilterRegistration.Dynamic delegatingFilterReg = servletContext.addFilter("springSecurityFilterChain", DelegatingFilterProxy.class);
	delegatingFilterReg.addMappingForUrlPatterns(null, true, "/*");

	...

}
```

---

## SecurityConfig.java 파일 생성

```java
// SecurityConfig.java

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

  /*
  * user authentication manager
  *  <security:authentication-manager>
  *   <security:authentication-provider>
  *       <security:user-service>
  *           <security:user name="user" password="password" authorities="ROLE_USER" />
  *       </security:user-service>
  *   </security:authentication-provider>
  *	</security:authentication-manager>
  */
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  	auth.inMemoryAuthentication()
        .withUser("user").password("password").roles("USER")
        .and().withUser("admin").password("admin").roles("ADMIN");
  }

  /**
  * <security:http pattern="/resources/**" security="none" />
  */
  @Override
  public void configure(WebSecurity web) throws Exception {
    web.ignoring()
       .antMatchers("/resources/**");
  }

 /**
  * <security:http auto-config="true">
  *
  *   <security:intercept-url pattern="/favicon.ico" access="permitAll" />
  *   <security:intercept-url pattern="/" access="permitAll" />
  *   <security:intercept-url pattern="/admin/**" access="hasRole('ADMIN')" />
  *   <security:intercept-url pattern="/**" access="IS_AUTHENTICATED_ANONYMOUSLY />
  *
  *   <security:csrf disabled="false"/>
  *
  *   <security:form-login
  *    login-page="/security/loginPage"
  *    login-processing-url="/login"
  *    username-parameter="username"
  *    password-parameter="password"
  *    default-target-url="/"/>
  *
  *   <security:logout logout-url="/logout" logout-success-url="/"/>
  *   <security:access-denied-handler error-page="/security/error"/>
  * </security:http>
  */    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http
        //.csrf().disable() // csrf 관련 설정 (보안관련 설정)
          .authorizeRequests()
          .antMatchers("/favicon.ico", "/").permitAll() // 모든 권한 허용
          .antMatchers("/admin/**").hasAnyRole("ADMIN") // "ADMIN"권한 허용
          .anyRequest().authenticated() // 여타 다른주소들은 "USER"권한 체크
          .and()
          .formLogin()  // 로그인 form
        //.usernameParameter("username") // form내 권한명 파라미터 name (default : username)
        //.passwordParameter("password") // form내 비밀번호 파라미터 name (default : password)
          .loginPage("/security/loginPage") // 권한이 필요한 페이지 접근시 로그인 페이지로 리다이렉
          .loginProcessingUrl("/login") // 로그인 url
          .permitAll()  // 로그인 페이지는 모든 권한 허용
          .and()
          .logout() // 로그아웃 설정
          .logoutUrl("/logout") // 로그아웃 url
          .logoutSuccessUrl("/") // 로그아웃 성공후 이동 url
          .permitAll() // 로그아웃 모든 권한 허용
          .and()
          .exceptionHandling().accessDeniedPage("/security/error"); // 권한 획득 관련 에러시 이동 url
    }
}
```

>[참고]
CSRF (Cross Site Request Forgery)
- 웹사이트의 취약점을 이용한 공격으로 사용자의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 하는 공격을 말한다.
Security 설정중 이와 관련된 옵션을 설정할 수 있다.
[참고 wiki](https://en.wikipedia.org/wiki/Cross-site_request_forgery)


---

## loginPage.jsp
SecurityConfig.java 에서 설정한 `loginProcessingUrl` 값을 form action으로 설정하고, `usernameParameter`, `passwordParameter` 값을 각각 form의 input name으로 추가

```html
<html>
<head>
	<title>Login</title>
</head>
<body>
<h1>
	Login!
</h1>
<br>
<form action="/login" method="POST">
 <table>
    <tbody><tr><td>User:</td><td><input type="text" name="username"></td></tr>
    <tr><td>Password:</td><td><input type="password" name="password"></td></tr>
    <tr><td colspan="2"><input name="submit" type="submit" value="Login"></td></tr>
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
  </tbody></table>
</form>
</body>
</html>
```
