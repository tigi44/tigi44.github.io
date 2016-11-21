---
layout: post
categories: "spring"
title: "[SPRING] JAVA CONFIG"
description: "Spring framework - java config"
modified: 2016-11-21
tags: [spring framwork, java config, spring4, java1.8, develop]
---

# JAVA CONFIG
Spring MVC 프로젝트에서 xml 파일 대신 java를 이용하여 설정할 수 있도록 환경 셋팅

* Spring MVC 프로젝트 생성 필요

<br>

# GIT SOURCE
[springStudy - java config commit](https://github.com/onlytigi/springStudy/commit/ed3baabe878b17e824895cf86a4acf160a3dd702)

<br>

# VERSION
___
- JAVA Version : 1.8
- Spring Framework Version : 4.x.x

___

<br>

# MAVEN 설정
Spring Framework 관련 maven 설정 필요

```xml
<!-- web.xml을 삭제하고 나오는 오류 메세지 해결 -->
<properties>
	<failOnMissingWebXml>false</failOnMissingWebXml>
</properties>

<!-- Spring -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>${org.springframework-version}</version>
	<exclusions>
		<!-- Exclude Commons Logging in favor of SLF4j -->
		<exclusion>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>${org.springframework-version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>${org.springframework-version}</version>
</dependency>
```
**[주의]**
<br>
web.xml 파일을 삭제하고 pom.xml에서 failOnMissingWebXml관련 오류가 발생할 경우 아래와 같은 설정을 pom.xml에 작성

```xml
<properties>
	<failOnMissingWebXml>false</failOnMissingWebXml>
</properties>
```

<br>

# 기존 XML 파일 삭제
- appServlet > root-context.xml 삭제
- spring > appServlet > servlet-context.xml 삭제
- web.xml 삭제

![project_before](/images/post/spring/javaconfig_project_before.png)

<br>

# JAVA CONFIG 파일 생성
![project_after](/images/post/spring/javaconfig_project_after.png)

- RootAppConfig.java : root-context.xml 대체 생성

```java
// RootAppConfig.java

@Configuration
@Import({})
@ComponentScan(
basePackages = {"com.onlytigi.springStudy"},
includeFilters = @Filter(type = FilterType.ANNOTATION, value = {Service.class}))

public class RootAppConfig implements InitializingBean {    
    @Override
    public void afterPropertiesSet() throws Exception {

    }
}
```

- SpringMvcConfig.java : servlet-context.xml 대체 생성

```xml
<!-- servlet-context.xml -->

<!-- Enables the Spring MVC @Controller programming model -->
<annotation-driven />

<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
<resources mapping="/resources/**" location="/resources/" />

<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<beans:property name="prefix" value="/WEB-INF/views/" />
	<beans:property name="suffix" value=".jsp" />
</beans:bean>

<context:component-scan base-package="com.onlytigi.springStudy" />
```

```java
// SpringMvcConfig.java

@Configuration
@EnableWebMvc
@EnableAspectJAutoProxy
@ComponentScan(basePackages = {"com.onlytigi.springStudy"},
		includeFilters = @Filter(value = {Controller.class, ControllerAdvice.class}),
		useDefaultFilters = false)

public class SpringMvcConfig extends WebMvcConfigurerAdapter {

	@Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/resources/**").addResourceLocations("/resources/").resourceChain(true);
  }

	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}

	@Override
	public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
		configurer.favorPathExtension(true)
							.useJaf(false)
							.ignoreAcceptHeader(false)
							.defaultContentType(MediaType.TEXT_HTML);
	}

	@Bean
	public ViewResolver contentNegotiatingViewResolver(ContentNegotiationManager manager) {
		List<ViewResolver> viewResolvers = new ArrayList<ViewResolver>();

		InternalResourceViewResolver defaultViewResolver = new InternalResourceViewResolver();
		defaultViewResolver.setPrefix("/WEB-INF/views/");
		defaultViewResolver.setSuffix(".jsp");
		defaultViewResolver.setViewClass(JstlView.class);

		viewResolvers.add(defaultViewResolver);

    ContentNegotiatingViewResolver contentViewResolver = new ContentNegotiatingViewResolver();
    contentViewResolver.setViewResolvers(viewResolvers);
    contentViewResolver.setContentNegotiationManager(manager);
    contentViewResolver.setOrder(0);
    return contentViewResolver;
	}

}

```

- WebConfig.java : web.xml 대체 생성


```xml
<!-- web.xml -->

<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/spring/root-context.xml</param-value>
</context-param>

<!-- Creates the Spring Container shared by all Servlets and Filters -->
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<!-- Encoding Filter -->
<filter>
	<filter-name>encodingFilter</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>encodingFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>

<!-- Processes application requests -->
<servlet>
	<servlet-name>appServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>appServlet</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

```java
// WebConfig.java

public class WebConfig implements WebApplicationInitializer {

	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		AnnotationConfigWebApplicationContext rootAppContext = new AnnotationConfigWebApplicationContext();
		rootAppContext.register(RootAppConfig.class);
		servletContext.addListener(new ContextLoaderListener(rootAppContext));

		AnnotationConfigWebApplicationContext springMvcContext = new AnnotationConfigWebApplicationContext();
		springMvcContext.register(SpringMvcConfig.class);

		FilterRegistration.Dynamic filterReg = servletContext.addFilter("encodingFilter", CharacterEncodingFilter.class);
		filterReg.setInitParameter("encoding", "utf-8");
		filterReg.setInitParameter("forceEncoding", "true");
		filterReg.addMappingForUrlPatterns(null, true, "/*");

		ServletRegistration.Dynamic dispatcherServlet = servletContext.addServlet("dispatcherServlet", new DispatcherServlet(springMvcContext));
		dispatcherServlet.setLoadOnStartup(1);
		dispatcherServlet.addMapping("/");
	}
}
```
<br>

**[주의]**
<br>
javax.servlet.ServletContext 에서 addListener 가 보이지 않는다면 pom.xml의 javax.servlet의 버전을 수정하여 해결할 수 있다.

```xml
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<version>3.0.1</version>
	<scope>provided</scope>
</dependency>
```
