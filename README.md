# Spring Boot Monitoring
## Server 설정

### 1. `pom.xml` 파일에 아래와 같이 추가
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.3.1</version>
</dependency>
```
### 2. `SecurityConfig` 파일을 만들고 아래와 같이 수정
```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final AdminServerProperties adminServer;

    public SecurityConfig(AdminServerProperties adminServer) {
        this.adminServer = adminServer;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // @formatter:off
        SavedRequestAwareAuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter("redirectTo");
        successHandler.setDefaultTargetUrl(this.adminServer.path("/"));

        http.authorizeRequests()
                .antMatchers(this.adminServer.path("/assets/**")).permitAll() // <1>
                .antMatchers(this.adminServer.path("/login")).permitAll()
                .anyRequest().authenticated() // <2>
                .and()
                .formLogin().loginPage(this.adminServer.path("/login")).successHandler(successHandler).and() // <3>
                .logout().logoutUrl(this.adminServer.path("/logout")).and()
                .httpBasic().and() // <4>
                .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()) // <5>
                .ignoringAntMatchers(
                        this.adminServer.path("/instances"), // <6>
                        this.adminServer.path("/actuator/**") // <7>
                );
        // @formatter:on
    }
}
```
### 3. `application.properties`에 아래와 같이 추
```
# 클라이언트에서 접속할때 사용해야 함..
spring.security.user.name=leebc
spring.security.user.password=1234
```

## 클라이언트 설정
### 1. `pom.xml` 파일에 아래와 같이 추가
```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.3.1</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 2. `SecurityConfig` 파일을 만들고 아래 내용 추가
```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().permitAll()
                .and().csrf().disable()
        ;
    }
}
```
### 3. `application.properties`에 아래와 같이 추가
```
# Admin 서버의 url과 접속 계정
spring.boot.admin.client.url=http://localhost:8080
spring.boot.admin.client.username=leebc
spring.boot.admin.client.password=1234
management.endpoints.web.exposure.include=*
```