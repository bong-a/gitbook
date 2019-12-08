# Spring Swagger 2 Guide

-------------

[TOC]

---------------



## 1. 설정하기

### 1. pom.xml

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

- ISSUE : [Failed to start bean 'documentationPluginsBootstrapper'](https://github.com/springfox/springfox/issues/2616#)

  - 해결방법 

  ```xml
  <dependency>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
      <version>20.0</version>
      <exclusions>
          <exclusion>
              <groupId>com.google.guava</groupId>
              <artifactId>guava</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

  

### 2. create SwaggerConfig.java

- `.apis()`: 해당 컨트롤러만 문서화하도록 basePackage을 `kr.datasolution.bigstation`로 설정한다.

  - *any()*, *none(),* *regex()*, or *ant()*. 등으로 여러 설정 가능 

    [참고]: https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api	"참고"

    

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.project.toy"))
                .paths(PathSelectors.any())
                .build();
    }
}
```

### 3. Swagger UI  붙이기

spring boot의 경우 자동으로 configuration을 해주지만 swagger ui의 경우에는 `WebMvcConfigurerAdapter`를 상속하는 클래스를 설정으로 만들어줘야만 한다.

또한 `@EnableWebMvc` 어노테이션도 있어야한다.

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/WEB-INF/resources/");

        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```

```java
@EnableWebMvc
@SpringBootApplication
public class WorkflowRuleClassifierApiApplication{
    
}
```



## 4. URL 접속 확인

```http
http://localhost:port/swagger-ui.html
```



## 2. 부가 설명 추가하기

### 1. swagger 메인화면에 설명 추가하기

```java
public class SwaggerConfig {
    //...
    //ADD
    private ApiInfo apiInfo() {
        return new ApiInfo(
                "Rule Classifier API",
                "BIGstation v3.1 - rule classifier api ",
                "API TOS",
                "Terms of service",
                new Contact("Bong A Lim", null, "bong@datasolution.kr"),
                "License of API",
                "API license URL",
                Collections.emptyList());
    }
}
```



### 2. 컨트롤러 설명 추가하기

```java
@Api(value = "main", description = "main controller")
public class RuleClassifierController {
}
```



### 3. 각 API마다 설명 추가하기

```java
@ApiOperation(value = "쿼리(검색) 테스트 API")
@RequestMapping(value = "/search", method = RequestMethod.POST)
    public Map<String, Object> search(@RequestBody SearchRequestVo requestVo, User user) throws Exception {
        //...
        return result;
    }
```



- 더 자세히 설정도 가능하다

  ```java
   @ApiImplicitParams({
              @ApiImplicitParam(name = "title", value = "제목", required = true, dataType = "string", paramType = "query", defaultValue = ""),
              @ApiImplicitParam(name = "content", value = "내용", required = true, dataType = "string", paramType = "query", defaultValue = ""),
      })
  ```

  - 의견을 덧붙이자면, requestBody의 경우 대부분 VO일때가 많은데, dataType을 뭐로 설정해줘야할지 모르겠다.

    `@ApiImplicitParams`설정 안하면 자동으로 Vo객체 매핑해주는데 `@ApiImplicitParams`으로 Vo 객체 매핑을 시도하였으나 매핑 실패해서 `@ApiImplicitParams`설정을 안해주었다.

    VO 객체 매핑 성공하시면 공유부탁드립니다.

    [참조]: https://yookeun.github.io/java/2017/02/26/java-swagger/

    