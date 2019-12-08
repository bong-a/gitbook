# Swagger Aggregation

어플리케이션마다 swagger ui를 가지고있다. 

각 서버마다 떠있는 swagger ui를 통합하여 보려한다.

리서치 결과 swagger에서 aggregation 프로젝트를 제공하고있어 그를 적용해보았다.

 API Gateway가 있어 API gateway서버에서 swagger ui를 통합해보려한다.

API Gateway는 Spring Cloud Netflix를 사용하고 있다.



### 1. pom.xml

   ```xml
   <!--swagger-->
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

### 2. bean 등록
   ```java
     @EnableSwagger2
   public class BalancerApplication {
       @Bean
       UiConfiguration uiConfig() {
           return new UiConfiguration("validatorUrl", "list", "alpha", "schema",UiConfiguration.Constants.DEFAULT_SUBMIT_METHODS, false, true, 60000L);
       }
   }
   ```


### 3. swagger ui 설정

    ```java
    @EnableWebMvc
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

### 4. SwaggerAggregator Component 생성
   ```java
   
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.context.annotation.Primary;
   import org.springframework.stereotype.Component;
   import springfox.documentation.swagger.web.SwaggerResource;
   import springfox.documentation.swagger.web.SwaggerResourcesProvider;
   
   import java.util.ArrayList;
   import java.util.List;
   
   // @Primary : 기본으로 등록되는 빈을 뜻한다.
   @Component
   @Primary
   @EnableAutoConfiguration
   public class SwaggerAggregatorController implements SwaggerResourcesProvider {
       @Override
       public List<SwaggerResource> get() {
           List<SwaggerResource> resources = new ArrayList<>();
           SwaggerResource swaggerResource = new SwaggerResource();
           swaggerResource.setName("rule-manager");
           swaggerResource.setLocation("/api/v0.2/rule/manager/v2/api-docs");
           swaggerResource.setSwaggerVersion("2.0");
   
           resources.add(swaggerResource);
           return resources;
       }
   }
   ```
