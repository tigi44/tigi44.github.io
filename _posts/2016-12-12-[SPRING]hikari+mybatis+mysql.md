---
layout: post
categories: "spring"
title: "[SPRING] JDBC - HIKARI CP + MYBATIS + MYSQL"
description: "Spring framework - spring jdbc db접근을 통한 Security 권한 획득하기"
modified: 2016-12-13
tags: [spring framwork, java config, security config, hikari, mybatis, mysql, spring4, java1.8, develop]
---

## JDBC - HIKARI CP + MYBATIS + MYSQL
Spring Security를 통한 유저권한 획득을 db에 저장된 유저 정보를 통해 진행할수 있도록 수정

* hikari cp + mybatis + mysql 이용
* db connection pool은 Commons DBCP 대신 HIKARI CP 사용
* mysql 로컬 설치 및 실행 필요

---

## GIT SOURCE
* [springStudy - Spring JDBC commit](https://github.com/onlytigi/springStudy/commit/d631a3649f9363d22e54343dcc9ece45962720b0)

---

## VERSION
- JAVA Version : 1.8
- Spring Framework Version : 4.x.x

---

## 사전작업 (Properties Config)
db 접속정보와 설정값들을 저장해 놓을 properties 파일을 사용하기 위해 사전작업으로 Properties Config 설정 추가 작업 진행

```java
// PropertiesConfig.java

/**
 * Properties Config
 */
@Configuration
public class PropertiesConfig {

	public static final String PROJECT_PROPERTY_PATH = "classpath*:properties/**/*";

	@Bean
	public static PropertyPlaceholderConfigurer propertyPlaceholderConfigurer() throws IOException {
		PropertyPlaceholderConfigurer propertyConfigurer = new PropertyPlaceholderConfigurer();
		propertyConfigurer.setLocations(getProjectResources());
		return propertyConfigurer;
	}

	public static Resource[] getProjectResources() throws IOException {
        ResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();
        Resource[] commResource = patternResolver.getResources(PROJECT_PROPERTY_PATH);
        return commResource;
	}
}
```

```
// db.properties

#############################################################
#           DB
#############################################################
jdbc.driverClassName=com.mysql.jdbc.Driver

#default
validationQuery=SELECT 1

#JDBC
jdbc.minimumIdle=2
jdbc.maximumPoolSize=10
jdbc.connectionTimeout=300000
jdbc.autocommit=false

#datasource
datasource.cachePrepStmts=true
datasource.prepStmtCacheSize=250
datasource.prepStmtCacheSqlLimit=2048
datasource.useServerPrepStmts=true

datasource.dbConnUsrNm=root
datasource.dbConnPwd=1234
datasource.dbConnUrl=jdbc:mysql://127.0.0.1:3306/localtest

```

---

## MAVEN 설정 (pom.xml)
Spring Framework jdbc, hikari, mybatis, mysql 관련 maven 설정 필요

```xml
<!-- Jdbc -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>${org.springframework-version}</version>
</dependency>

<!-- DB Connection pool -->
<dependency>
	<groupId>com.zaxxer</groupId>
	<artifactId>HikariCP</artifactId>
	<version>2.3.2</version>
</dependency>

<!-- mybatis -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.2.3</version>
</dependency>
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>1.2.2</version>
</dependency>

<!-- mysql -->
<dependency>
	 <groupId>mysql</groupId>
	 <artifactId>mysql-connector-java</artifactId>
	 <version>5.1.34</version>
</dependency>

<!-- Cglib -->
<dependency>
	<groupId>org.sonatype.sisu.inject</groupId>
	<artifactId>cglib</artifactId>
	<version>2.2.2</version>
</dependency>
```
>[참고]
`cglib`은 트랙잭션 설정에서 프록시설정을 이용하기 위해 추가

---

## DataSourceConfig.java
mybatis, hikari data source 와 transaction manager 설정

```java
// DataSourceConfig.java

/**
 * DataSource 설정
 */
@Configuration
@EnableTransactionManagement(mode = AdviceMode.PROXY, order = 0)
//@MapperScan(basePackages = "com.onlytigi.**.dao", annotationClass = Repository.class)
@ComponentScan(basePackages = {"com.onlytigi.**.dao"},
	    	   includeFilters = @Filter(type = FilterType.ANNOTATION, value = {Repository.class}))

public class DataSourceConfig {

	// 미리 설정해둔 PropertiesConfig 설정으로 db.properties 파일내에 저장되어 있는 설정값을 읽어옴
	@Value("${jdbc.driverClassName}")
	private String driverClassName;

	@Value("${jdbc.minimumIdle}")
	private int minimumIdle;
	@Value("${jdbc.maximumPoolSize}")
	private int maximumPoolSize;
	@Value("${jdbc.connectionTimeout}")
	private int connectionTimeout;
	@Value("${validationQuery}")
	private String validationQuery;
	@Value("${jdbc.autocommit}")
	private boolean isAutoCommit;

	@Value("${datasource.cachePrepStmts}")
	private String cachePrepStmts;
	@Value("${datasource.prepStmtCacheSize}")
	private String prepStmtCacheSize;
	@Value("${datasource.prepStmtCacheSqlLimit}")
	private String prepStmtCacheSqlLimit;
	@Value("${datasource.useServerPrepStmts}")
	private String useServerPrepStmts;

	@Value("${datasource.dbConnUsrNm}")
	private String dbConnUsrNm;
	@Value("${datasource.dbConnPwd}")
	private String dbConnPwd;
	@Value("${datasource.dbConnUrl}")
	private String dbConnUrl;

	/**
	 * Hikari data source
	 */
	@Bean(destroyMethod = "shutdown")
	public HikariDataSource hikariDataSource() {
		HikariConfig config = getHikariConfig();
		config.setDriverClassName(driverClassName);
        config.addDataSourceProperty("user", dbConnUsrNm);
        config.addDataSourceProperty("password", dbConnPwd);
        config.setJdbcUrl(dbConnUrl);
	    return new HikariDataSource(config);
	}
	private HikariConfig getHikariConfig() {
		HikariConfig config = new HikariConfig();
        config.setMinimumIdle(minimumIdle);
        config.setMaximumPoolSize(maximumPoolSize);
        config.setConnectionTestQuery(validationQuery);
        config.setConnectionTimeout(connectionTimeout);
        config.setAutoCommit(isAutoCommit);

        config.addDataSourceProperty("cachePrepStmts", cachePrepStmts);
        config.addDataSourceProperty("prepStmtCacheSize", prepStmtCacheSize);
        config.addDataSourceProperty("useServerPrepStmts", useServerPrepStmts);
        return config;
	}

	/**
	 * Data source
	 */
	@Bean
	public DataSource dataSource() {
	    return new LazyConnectionDataSourceProxy(hikariDataSource());
	}

	/**
	 * Data source transaction manager
	 */
	@Bean
	public DataSourceTransactionManager transactionManager() {
		DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(dataSource());
		return dataSourceTransactionManager;
	}

	/**
	 * sql session factory
	 * - mybatis-config.xml path
	 * - mapper path
	 */
	@Bean
	public SqlSessionFactory sqlSessionFactory() throws Exception {
		SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
		sqlSessionFactory.setDataSource(dataSource());
		PathMatchingResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
		DefaultResourceLoader defaultResourceLoader = new DefaultResourceLoader();
		sqlSessionFactory.setConfigLocation(defaultResourceLoader.getResource("classpath:mybatis/mybatis-config.xml"));
		sqlSessionFactory.setMapperLocations(resourcePatternResolver.getResources("classpath*:mapper/**/*.xml"));

		// default TypeHandler 등록.
		TypeHandlerRegistry typeHandlerRegistry = sqlSessionFactory.getObject().getConfiguration().getTypeHandlerRegistry();
		typeHandlerRegistry.register(java.sql.Timestamp.class, org.apache.ibatis.type.DateTypeHandler.class);
		typeHandlerRegistry.register(java.sql.Time.class, org.apache.ibatis.type.DateTypeHandler.class);
		typeHandlerRegistry.register(java.sql.Date.class, org.apache.ibatis.type.DateTypeHandler.class);

		return sqlSessionFactory.getObject();
	}

	@Bean(destroyMethod = "clearCache")
	public SqlSessionTemplate sqlSession(SqlSessionFactory sqlSessionFactory) {
		SqlSessionTemplate sessionTemplate = new SqlSessionTemplate(sqlSessionFactory);
		return sessionTemplate;
	}
}
```

---

## mybatis-config.xml
mybatis 설정값

```xml
<!-- mybatis-config.xml -->

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <!-- default 옵션을 설정한다. -->
    <settings>
        <setting name="cacheEnabled" value="true"/>

        <!-- lazy로딩 수행여부 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="true"/>

        <!-- insert구문 사용시 GENERATED_KEYS 기능을 사용해야 한다면 true로 설정 기본값: false
             insert시 자동생성된 key를 추가 sql없이 얻을수 있음 -->
        <setting name="useGeneratedKeys" value="true"/>

        <!-- mybatis 의 defaultExecutorType를 설정한다 기본값 : SIMPLE-->
        <!-- dbcp에서 preparedStatement cache 를 수행하므로 REUSE를 사용할 필요는 없음. -->
        <setting name="defaultExecutorType" value="SIMPLE"/>

        <!--defaultQueryTiemout을 설정한다. 초단위 -->
        <setting name="defaultStatementTimeout" value="5"/>
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>
    <typeAliases>
    	<!-- <typeAlias alias="hMap" type="java.util.HashMap"/> -->
        <package name="com.onlytigi.springStudy.model"/>
    </typeAliases>
</configuration>
```
>[참고]
`<typeAliases>` 안에 `typeAlias`, `package`를 설정함으로  mapper sql에서 parameterType, resultType 경로를 따로 포함시키지 않아도 됨

---

## SecurityConfig.java
권한 획득을 요청한 유저 정보를 db에서 읽어오기 위해 기존 SecurityConfig.java 파일 수정

```java
// SecurityConfig.java

 /*
	* DB 접속을 통하여 사용자 정보 및 권한을 확인할 Service
	*/
	@Autowired UserAuthenticationService userAuthenticationService;

	...

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		// 기존 명시적으로 사용하던 부분 삭제
//		auth.inMemoryAuthentication() .withUser("user").password("password").roles("USER")
//		.and().withUser("admin").password("admin").roles("ADMIN");

		// db로 부터 유저정보를 읽어와서 해당 권한 체크 부분 추가
		auth.userDetailsService(userAuthenticationService)
		    .passwordEncoder(new ShaPasswordEncoder(256)); // 비밀번호 암호화
	}

	....
```

---

## UserDetailsService
db에서 유저 정보를 읽어와 권한을 체크하고, 유저정보를 보관하는 부분

```java
// UserAuthenticationService.java

/**
 * db로 부터 읽어온 해당 유저 권한 체크
 */
@Service
public class UserAuthenticationService implements UserDetailsService {

	@Autowired
	SecurityDao securityDao;

	// UserDetails는 spring의 User객체
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		User user = securityDao.selectUserInfo(username);
		if(user == null) throw new UsernameNotFoundException(username);
		List<GrantedAuthority> gas = new ArrayList<GrantedAuthority>();
		gas.add(new SimpleGrantedAuthority(user.getAuthority()));
		return new UserDetail(user.getId(), user.getPassword(), "JOIN".equalsIgnoreCase(user.getStat()), true, true, true, gas, user.getName(), user.getIdNo());
	}
}
```
>[참고] `User` 객체는 유저정보를 db에서 읽어올수 있도록 만든 객체

---

## DAO & Mapper  
db정보를 읽어오는 DAO와 Mapper

```java
// SecurityDao.java

/**
 * 유저 권한 정보를 db로 부터 읽어 올 DAO
 */
@Repository
public class SecurityDao {

    @Qualifier("sqlSession")
    @Autowired
    private SqlSessionTemplate sqlSession;

    public static final String MAPPER_NAMESPACE = "mapper.security.dao.";

    public User selectUserInfo(String id) {
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("id", id);
    	return sqlSession.selectOne(MAPPER_NAMESPACE + "selectUserInfo", param);
    }
```		

```xml
<!-- security_dao.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper.security.dao">

<select id="selectUserInfo" parameterType="map" resultType="user">
		SELECT
			*
		FROM
			user
		WHERE
			id = #{id}
</select>

</mapper>
```
