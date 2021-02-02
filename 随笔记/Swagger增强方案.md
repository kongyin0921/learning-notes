# Swagger 增强方案

内容：

1. SpringBoot 项目中如何使用？
2. Spring Security 项目中如何使用？
3. 使用 knife4j 增强 Swagger

## SpringBoot 项目中如何使用？

Swagger3.0 官方已经有了自己的 Spring Boot Starter，只需要添加一个 jar 包即可（SpringBoot 版本 2.3.6.RELEASE）。。

```xml
<!-- swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

什么都不用配置！直接在浏览器中访问 :http://ip:port/swagger-ui/ 即可。

## Spring Security 项目中如何使用？

如果你的项目使用了 Spring Security 做权限认证的话，你需要为 Swagger 相关 url 添加白名单。

```java
String[] SWAGGER_WHITELIST = {
            "/swagger-ui.html",
            "/swagger-ui/*",
            "/swagger-resources/**",
            "/v2/api-docs",
            "/v3/api-docs",
            "/webjars/**"
    };

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and()
                // 禁用 CSRF
                .csrf().disable()
                .authorizeRequests()
                // swagger
                .antMatchers(SWAGGER_WHITELIST).permitAll()
          ......
    }
```

另外，某些请求需要认证之后才可以访问，为此，我们需要对 Swagger 做一些简单的配置。

配置的方式非常简单，我提供两种不同的方式给小伙伴们。

1. 登录后自动为请求添加 token。
2. 为请求的 Header 添加一个认证参数，每次请求的时候，我们需要手动输入 token。

### 登录后自动为请求添加 token

通过这种方式我们只需要授权一次即可使用所有需要授权的接口。

```java
/**
 * @author shuang.kou
 * @description swagger 相关配置
 */
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("github.javaguide.springsecurityjwtguide"))
                .paths(PathSelectors.any())
                .build()
                .securityContexts(securityContext())
                .securitySchemes(securitySchemes());
    }

    private List<SecurityScheme> securitySchemes() {
        return Collections.singletonList(new ApiKey("JWT", SecurityConstants.TOKEN_HEADER, "header"));
    }

    private List<SecurityContext> securityContext() {
        SecurityContext securityContext = SecurityContext.builder()
                .securityReferences(defaultAuth())
                .build();
        return Collections.singletonList(securityContext);
    }

    List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope
                = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        return Collections.singletonList(new SecurityReference("JWT", authorizationScopes));
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Security JWT Guide")
                .build();
    }
}
```

### 为请求的 Header 添加一个认证参数

每次请求的时候，我们需要手动输入 token 到指定位置。

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("github.javaguide.springsecurityjwtguide"))
                .paths(PathSelectors.any())
                .build()
                .globalRequestParameters(authorizationParameter())
                .securitySchemes(securitySchemes());
    }

    private List<SecurityScheme> securitySchemes() {
        return Collections.singletonList(new ApiKey("JWT", SecurityConstants.TOKEN_HEADER, "header"));
    }

    private List<RequestParameter> authorizationParameter() {
        RequestParameterBuilder tokenBuilder = new RequestParameterBuilder();
        tokenBuilder
                .name("Authorization")
                .description("JWT")
                .required(false)
                .in("header")
                .accepts(Collections.singleton(MediaType.APPLICATION_JSON))
                .build();
        return Collections.singletonList(tokenBuilder.build());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Security JWT Guide")
                .build();
    }
}
```

## 使用 knife4j 增强 Swagger

1. **YApi** ：YApi 是一个可本地部署的、打通前后端及 QA 的、可视化的接口管理平台。可以帮助我们让 swagger 页面的体验更加友好，目前很多大公司都在使用这个开源工具。项目地址：https://github.com/YMFE/yapi 。使用方法：[当 Swagger 遇上 YApi，瞬间高大上了！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247496458&idx=1&sn=6f8786a995cc171e80d9350529cc70a9&chksm=cea1bcc1f9d635d70a9c3d221d5f630046af8cd5e0d0303be48ebe2a2367df7ce442faed7d8d&token=212861022&lang=zh_CN&scene=21#wechat_redirect)
2. **Knife4j** ：Swagger 生成 Api 文档的增强解决方案，前身是 swagger-bootstrap-ui 。官方文档：https://xiaoym.gitee.io/knife4j/documentation/ 。

根据官网介绍，knife4j 是为 Java MVC 框架集成 Swagger 生成 Api 文档的增强解决方案。

项目地址：https://gitee.com/xiaoym/knife4j 。

使用方式非常简单,添加到相关依赖即可（SpringBoot 版本 2.3.6.RELEASE）。

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```

完成之后，访问：http://ip:port/doc.html 即可。

效果如下。可以看出，相比于 swagger 原生 ui 确实好看实用了很多。

除了 UI 上的增强之外，knife4j 还提供了一些开箱即用的功能。

比如：**搜索 API 接口** （`knife4j` 版本>2.0.1 ）

再比如：**导出离线文档**

通过 `Knife4j` 我们可以非常方便地导出 Swagger 文档 ，并且支持多种格式。

> - markdown：导出当前逻辑分组下所有接口的 Markdown 格式的文档
> - Html：导出当前逻辑分组下所有接口的 Html 格式的文档
> - Word:导出当前逻辑分组下所有接口的 Word 格式的文档(自 2.0.5 版本开始)
> - OpenAPI:导出当前逻辑分组下的原始 OpenAPI 的规范 json 结构(自 2.0.6 版本开始)
> - PDF:未实现

以 HTML 格式导出