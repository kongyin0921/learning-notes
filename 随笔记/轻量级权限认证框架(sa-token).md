# 轻量级权限认证框架(sa-token)

## Sa-Token是什么？

Sa-Token是一个轻量级Java权限认证框架，主要解决：登录认证、权限认证、Session会话、单点登录、OAuth2.0 等一系列权限相关问题

框架针对踢人下线、自动续签、前后台分离、分布式会话……等常见业务进行N多适配，通过sa-token，你可以以一种极简的方式实现系统的权限认证部分

与其它权限认证框架相比，`sa-token` 具有以下优势：

1. **简单** ：可零配置启动框架，真正的开箱即用，低成本上手
2. **强大** ：目前已集成几十项权限相关特性，涵盖了大部分业务场景的解决方案
3. **易用** ：如丝般顺滑的API调用，大量高级特性统统只需一行代码即可实现
4. **高扩展** ：几乎所有组件都提供了扩展接口，90%以上的逻辑都可以按需重写

有了sa-token，你所有的权限认证问题，都不再是问题！

## Springboot集成

- 依赖

  ```xml
  <!-- Sa-Token 权限认证, 在线文档：http://sa-token.dev33.cn/ -->
  <dependency>
      <groupId>cn.dev33</groupId>
      <artifactId>sa-token-spring-boot-starter</artifactId>
      <version>1.19.0</version>
  </dependency>
  ```

- 配置

  ```yml
  server:
      # 端口
      port: 8081
  
  spring: 
      # sa-token配置
      sa-token: 
          # token名称 (同时也是cookie名称)
          token-name: satoken
          # token有效期，单位s 默认30天, -1代表永不过期 
          timeout: 2592000
          # token临时有效期 (指定时间内无操作就视为token过期) 单位: 秒
          activity-timeout: -1
          # 是否允许同一账号并发登录 (为true时允许一起登录, 为false时新登录挤掉旧登录) 
          allow-concurrent-login: false
          # 在多人登录同一账号时，是否共用一个token (为true时所有登录共用一个token, 为false时每次登录新建一个token) 
          is-share: false
          # token风格
          token-style: uuid
          # 是否输出操作日志 
          is-log: false
  ```

## 基础

### 登录认证

- 登录注销
```java
// 标记当前会话登录的账号id 
// 建议的参数类型：long | int | String， 不可以传入复杂类型，如：User、Admin等等
StpUtil.setLoginId(Object loginId);    

// 当前会话注销登录
StpUtil.logout();

// 获取当前会话是否已经登录，返回true=已登录，false=未登录
StpUtil.isLogin();

// 检验当前会话是否已经登录, 如果未登录，则抛出异常：`NotLoginException`
StpUtil.checkLogin()
```
- 会话查询
```java
// 获取当前会话登录id, 如果未登录，则抛出异常：`NotLoginException`
StpUtil.getLoginId();

// 类似查询API还有：
StpUtil.getLoginIdAsString();    // 获取当前会话登录id, 并转化为`String`类型
StpUtil.getLoginIdAsInt();       // 获取当前会话登录id, 并转化为`int`类型
StpUtil.getLoginIdAsLong();      // 获取当前会话登录id, 并转化为`long`类型

// ---------- 指定未登录情形下返回的默认值 ----------

// 获取当前会话登录id, 如果未登录，则返回null 
StpUtil.getLoginIdDefaultNull();

// 获取当前会话登录id, 如果未登录，则返回默认值 （`defaultValue`可以为任意类型）
StpUtil.getLoginId(T defaultValue);
```
- 其他API

```java
// 获取指定token对应的登录id，如果未登录，则返回 null
StpUtil.getLoginIdByToken(String tokenValue);

// 获取当前`StpLogic`的token名称
StpUtil.getTokenName();

// 获取当前会话的token值
StpUtil.getTokenValue();

// 获取当前会话的token信息参数
StpUtil.getTokenInfo();
```

### 权限认证

> 1. 如何获取一个账号所拥有的的权限码集合
> 2. 本次操作需要验证的权限码是哪个

- 获取当前账号的权限集合

  根据自己的业务逻辑进行重写实现`StpInterface`接口，例如以下代码：

  ```java
  package com.pj.satoken;
  
  import java.util.ArrayList;
  import java.util.List;
  import org.springframework.stereotype.Component;
  import cn.dev33.satoken.stp.StpInterface;
  
  /**
   * 自定义权限验证接口扩展 
   */
  @Component    // 保证此类被SpringBoot扫描，完成sa-token的自定义权限验证扩展 
  public class StpInterfaceImpl implements StpInterface {
  
      /**
       * 返回一个账号所拥有的权限码集合 
       */
      @Override
      public List<String> getPermissionList(Object loginId, String loginKey) {
          // 本list仅做模拟，实际项目中要根据具体业务逻辑来查询权限
          List<String> list = new ArrayList<String>();    
          list.add("101");
          list.add("user-add");
          list.add("user-delete");
          list.add("user-update");
          list.add("user-get");
          list.add("article-get");
          return list;
      }
  
      /**
       * 返回一个账号所拥有的角色标识集合 (权限与角色可分开校验)
       */
      @Override
      public List<String> getRoleList(Object loginId, String loginKey) {
          // 本list仅做模拟，实际项目中要根据具体业务逻辑来查询角色
          List<String> list = new ArrayList<String>();    
          list.add("admin");
          list.add("super-admin");
          return list;
      }
  
  }
  
  ```

- 权限认证

  ```java
  // 当前账号是否含有指定权限, 返回true或false 
  StpUtil.hasPermission("user-update");        
  
  // 当前账号是否含有指定权限, 如果验证未通过，则抛出异常: NotPermissionException 
  StpUtil.checkPermission("user-update");        
  
  // 当前账号是否含有指定权限 [指定多个，必须全部验证通过] 
  StpUtil.checkPermissionAnd("user-update", "user-delete");        
  
  // 当前账号是否含有指定权限 [指定多个，只要其一验证通过即可] 
  StpUtil.checkPermissionOr("user-update", "user-delete");        
  ```

- 角色认证

  > 在sa-token中，角色和权限可以独立验证

  ```java
  // 当前账号是否含有指定角色标识, 返回true或false 
  StpUtil.hasRole("super-admin");        
  
  // 当前账号是否含有指定角色标识, 如果验证未通过，则抛出异常: NotRoleException 
  StpUtil.checkRole("super-admin");        
  
  // 当前账号是否含有指定角色标识 [指定多个，必须全部验证通过] 
  StpUtil.checkRoleAnd("super-admin", "shop-admin");        
  
  // 当前账号是否含有指定角色标识 [指定多个，只要其一验证通过即可] 
  StpUtil.checkRoleOr("super-admin", "shop-admin");        
  ```

- 拦截全局异常

  有同学要问，鉴权失败，抛出异常，然后呢？要把异常显示给用户看吗？**当然不可以！**
  你可以创建一个全局异常拦截器，统一返回给前端的格式

  ```java
  
  ```

- 权限通配符

  > Sa-Token允许你根据通配符指定泛权限，例如当一个账号拥有`user*`的权限时，`user-add`、`user-delete`、`user-update`都将匹配通过

  ```java
  // 当拥有 user* 权限时
  StpUtil.hasPermission("user-add");        // true
  StpUtil.hasPermission("user-update");     // true
  StpUtil.hasPermission("art-add");         // false
  
  // 当拥有 *-delete 权限时
  StpUtil.hasPermission("user-add");        // false
  StpUtil.hasPermission("user-delete");     // true
  StpUtil.hasPermission("art-delete");      // true
  
  // 当拥有 *.js 权限时
  StpUtil.hasPermission("index.js");        // true
  StpUtil.hasPermission("index.css");       // false
  StpUtil.hasPermission("index.html");      // false
  ```

  > 上帝权限：当一个账号拥有 `"*"` 权限时，他可以验证通过任何权限码 (角色认证同理)

### Session会话

Session是会话中专业的数据缓存组件，通过`Session`我们可以很方便的缓存一些高频读写数据，提高程序性能
在`sa-token`中, `Session` 分为三种, 分别是：

- `User-Session`: 指的是框架为每个`loginId`分配的`Session`

  ```java
  // 获取当前账号id的Session (必须是登录后才能调用)
  StpUtil.getSession();
  
  // 获取当前账号id的Session, 并决定在Session尚未创建时，是否新建并返回
  StpUtil.getSession(true);
  
  // 获取账号id为10001的Session
  StpUtil.getSessionByLoginId(10001);
  
  // 获取账号id为10001的Session, 并决定在Session尚未创建时，是否新建并返回
  StpUtil.getSessionByLoginId(10001, true);
  
  // 获取SessionId为xxxx-xxxx的Session, 在Session尚未创建时, 返回null 
  StpUtil.getSessionBySessionId("xxxx-xxxx");
  
  ```

- `Token-Session`: 指的是框架为每个`token`分配的`Session`

  ```java
  // 获取当前token的专属Session 
  StpUtil.getTokenSession();
  
  // 获取指定token的专属Session 
  StpUtil.getTokenSessionByToken(token);
  ```

  > 在未登录状态下是否可以获取`Token-Session`？这取决于你配置的`tokenSessionCheckLogin`值是否为false

- `自定义Session`: 指的是以一个`特定的值`作为SessionId，来分配的`Session`

  ```java
  // 查询指定key的Session是否存在
  SaSessionCustomUtil.isExists("goods-10001");
  
  // 获取指定key的Session，如果没有，则新建并返回
  SaSessionCustomUtil.getSessionById("goods-10001");
  
  // 获取指定key的Session，如果没有，第二个参数决定是否新建并返回  
  SaSessionCustomUtil.getSessionById("goods-10001", false);   
  
  // 删除指定key的Session
  SaSessionCustomUtil.deleteSessionById("goods-10001");
  ```

- Session相关操作

  ```java
  // 返回此Session的id 
  session.getId();                          
  
  // 返回此Session的创建时间 (时间戳) 
  session.getCreateTime();                  
  
  // 在Session上获取一个值 
  session.getAttribute('name');             
  
  // 在Session上获取一个值，并指定取不到值时返回的默认值
  session.getAttribute('name', 'zhang');    
  
  // 在Session上写入一个值 
  session.setAttribute('name', 'zhang');    
  
  // 在Session上移除一个值 
  session.removeAttribute('name');          
  
  // 清空此Session的所有值 
  session.clearAttribute();                 
  
  // 获取此Session是否含有指定key (返回true或false)
  session.containsAttribute('name');        
  
  // 获取此Session会话上所有key (返回Set<String>)
  session.attributeKeys();                  
  
  // 返回此Session会话上的底层数据对象（如果更新map里的值，请调用session.update()方法避免产生脏数据）
  session.getDataMap();                     
  
  // 将这个Session从持久库更新一下
  session.update();                         
  
  // 注销此Session会话 (从持久库删除此Session)
  session.logout();                         
  ```

- 类型转换`API`

  ```java
  // 写值 
  session.set("name", "zhang"); 
  
  // 写值 (只有在此key原本无值的时候才会写入)
  session.setDefaultValue("name", "zhang");
  
  // 取值
  session.get("name");
  
  // 取值 (指定默认值)
  session.get("name", "<defaultValue>"); 
  
  // 取值 (转String类型)
  session.getString("name"); 
  
  // 取值 (转int类型)
  session.getInt("age"); 
  
  // 取值 (转long类型)
  session.getLong("age"); 
  
  // 取值 (转double类型)
  session.getDouble("result"); 
  
  // 取值 (转float类型)
  session.getFloat("result"); 
  
  // 取值 (指定转换类型)
  session.getModel("key", Student.class); 
  
  // 取值 (指定转换类型, 并指定值为Null时返回的默认值)
  session.getModel("key", Student.class, <defaultValue>); 
  
  // 是否含有某个key
  session.has("key"); 
  ```

- Session环境隔离

  ```java
  @PostMapping("/resetPoints")
  public void reset(HttpSession session) {
      // 在HttpSession上写入一个值 
      session.setAttribute("name", 66);
      // 在SaSession进行取值
      System.out.println(StpUtil.getSession().getAttribute("name"));    // 输出null
  }
  ```

  > **要点：**
  >
  > - `SaSession` 与 `HttpSession` 没有任何关系，在`HttpSession`上写入的值，在`SaSession`中无法取出
  >
  > - `HttpSession`并未被框架接管，在使用sa-token时，请在任何情况下均使用`SaSession`，不要使用`HttpSession`

### 踢人下线

> 所谓踢人下线，核心操作就是找到其指定`loginId`对应的`token`，并设置其失效

- 根据账号id踢人下线

  ```java
  // 使账号id为10001的会话注销登录（踢人下线），待到10001再次访问系统时会抛出`NotLoginException`异常，场景值为-5
  StpUtil.logoutByLoginId(10001); 
  ```

- 根据token替人下线

  ```java
  // 使账号id为10001的会话注销登录
  StpUtil.logoutByTokenValue("xxxx-xxxx-xxxx-xxxx-xxxx");
  ```

  >此方法直接删除了`token->uid`的映射关系，对方再次访问时提示:`token无效`，场景值为-2

- 账号封禁

  > 对于违规账号，需要对其进行**账号封禁**防止其再次登录

  ```java
  // 封禁指定账号 
  // 参数一：账号id
  // 参数二：封禁时长，单位：秒  (86400秒=1天，此值为-1时，代表永久封禁)
  StpUtil.disable(10001, 86400); 
  
  // 获取指定账号是否已被封禁 (true=已被封禁, false=未被封禁) 
  StpUtil.isDisable(10001); 
  
  // 获取指定账号剩余封禁时间，单位：秒
  StpUtil.getDisableTime(10001); 
  
  // 解除封禁
  StpUtil.untieDisable(10001); 
  ```

- 注意

  ```java
  // 先踢下线
  StpUtil.logoutByLoginId(10001); 
  // 再封禁账号
  StpUtil.disableLoginId(10001, 86400); 
  ```

### 注解式鉴权

```java
@SaCheckLogin//标注在方法或类上，当前会话必须处于登录状态才可通过校验

@SaCheckRole("admin")//标注在方法或类上，当前会话必须具有指定角色标识才能通过校验

@SaCheckPermission("user:add")// 标注在方法或类上，当前会话必须具有指定权限才能通过校验
```

Sa-Token使用全局拦截器完成注解鉴权功能，为了不为项目带来不必要的性能负担，拦截器默认处于关闭状态
因此，为了使用注解鉴权，你必须手动将sa-token的全局拦截器注册到你项目中

- 注册拦截器

  以`SpringBoot2.0`为例, 新建配置类`SaTokenConfigure.java`

  ```java
  @Configuration
  public class SaTokenConfigure implements WebMvcConfigurer {
      // 注册sa-token的注解拦截器，打开注解式鉴权功能 
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          registry.addInterceptor(new SaAnnotationInterceptor()).addPathPatterns("/**");    
      }
  }
  ```

- 使用注解鉴权

  ```java
  // 登录认证：当前会话必须登录才能通过 
  @SaCheckLogin                        
  @RequestMapping("info")
  public String info() {
      return "查询用户信息";
  }
  
  // 角色认证：当前会话必须具有指定角色标识才能通过 
  @SaCheckRole("super-admin")        
  @RequestMapping("add")
  public String add() {
      return "用户增加";
  }
  
  // 权限认证：当前会话必须具有指定权限才能通过 
  @SaCheckPermission("user-add")        
  @RequestMapping("add")
  public String add() {
      return "用户增加";
  }
  ```

  > 注：以上注解都可以加在类上，代表为这个类所有方法进行鉴权

- 设定校验模式

  `@SaCheckRole`与`@SaCheckPermission`注解可设置校验模式，例如：

  ```java
  // 注解式鉴权：只要具有其中一个权限即可通过校验 
  @RequestMapping("atJurOr")
  @SaCheckPermission(value = {"user-add", "user-all", "user-delete"}, mode = SaMode.OR)        
  public AjaxJson atJurOr() {
      return AjaxJson.getSuccessData("用户信息");
  }
  ```

  mode有两种取值：

  - `SaMode.AND`, 标注一组权限，会话必须全部具有才可通过校验
  - `SaMode.OR`, 标注一组权限，会话只要具有其一即可通过校验

- 在业务逻辑使用注解鉴权

  以上拦截模式仅支持controller层的拦截，添加以下依赖开启`AOP`模式即可任意层级使用注解鉴权

  ```xml
  <!-- sa-token整合SpringAOP实现注解鉴权 -->
  <dependency>
      <groupId>cn.dev33</groupId>
      <artifactId>sa-token-spring-aop</artifactId>
      <version>1.19.0</version>
  </dependency>
  ```

  > 使用拦截器模式，只能把注解写在`Controller层`，使用`AOP`模式，可以将注解写在任意层级
  >
  > **拦截器模式和`AOP`模式不可同时集成**，否则会在`Controller层`发生一个注解校验两次的bug

### 路由拦截式鉴权

> 项目中所有接口均需要登录验证，只有'登录接口'本身对外开放

我们怎么实现呢？给每个接口加上鉴权注解？手写全局拦截器？似乎都不是非常方便。
在这个需求中我们真正需要的是一种基于路由拦截的鉴权模式, 那么在sa-token怎么实现路由拦截鉴权呢？

- 注册路由拦截器

  以`SpringBoot2.0`为例, 新建配置类`SaTokenConfigure.java`

  ```java
  @Configuration
  public class SaTokenConfigure implements WebMvcConfigurer {
      // 注册sa-token的登录拦截器
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          // 注册登录拦截器，并排除登录接口地址 
          registry.addInterceptor(new SaRouteInterceptor()).addPathPatterns("/**").excludePathPatterns("/user/doLogin"); 
      }
  }
  ```

  > 以上代码，我们注册了一个登录验证拦截器，并且排除了`/user/doLogin`接口用来开放登录（除了`/user/doLogin`以外的所有接口都需要登录才能访问）

- 自定义权限验证规则

  你可以使用函数式编程自定义验证规则

  ```java
  @Configuration
  public class SaTokenConfigure implements WebMvcConfigurer {
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          // 注册路由拦截器，自定义验证规则 
          registry.addInterceptor(new SaRouteInterceptor((request, response, handler)->{
              // 根据路由划分模块，不同模块不同鉴权 
              SaRouterUtil.match("/user/**", () -> StpUtil.checkPermission("user"));
              SaRouterUtil.match("/admin/**", () -> StpUtil.checkPermission("admin"));
              SaRouterUtil.match("/goods/**", () -> StpUtil.checkPermission("goods"));
              SaRouterUtil.match("/orders/**", () -> StpUtil.checkPermission("orders"));
              SaRouterUtil.match("/notice/**", () -> StpUtil.checkPermission("notice"));
              SaRouterUtil.match("/comment/**", () -> StpUtil.checkPermission("comment"));
          })).addPathPatterns("/**");
      }
  }
  ```

- 完整实例

  ```java
  @Configuration
  public class SaTokenConfigure implements WebMvcConfigurer {
      // 注册sa-token的拦截器
      @Override
      public void addInterceptors(InterceptorRegistry registry) {
          // 注册路由拦截器，自定义验证规则 
          registry.addInterceptor(new SaRouteInterceptor((request, response, handler) -> {
  
              // 登录验证 -- 拦截所有路由，并排除/user/doLogin 用于开放登录 
              SaRouterUtil.match("/**", "/user/doLogin", () -> StpUtil.checkLogin());
  
              // 登录验证 -- 排除多个路径
              SaRouterUtil.match(Arrays.asList("/**"), Arrays.asList("/user/doLogin", "/user/reg"), () -> StpUtil.checkLogin());
  
              // 角色认证 -- 拦截以 admin 开头的路由，必须具备[admin]角色或者[super-admin]角色才可以通过认证 
              SaRouterUtil.match("/admin/**", () -> StpUtil.checkRoleOr("admin", "super-admin"));
  
              // 权限认证 -- 不同模块, 校验不同权限 
              SaRouterUtil.match("/user/**", () -> StpUtil.checkPermission("user"));
              SaRouterUtil.match("/admin/**", () -> StpUtil.checkPermission("admin"));
              SaRouterUtil.match("/goods/**", () -> StpUtil.checkPermission("goods"));
              SaRouterUtil.match("/orders/**", () -> StpUtil.checkPermission("orders"));
              SaRouterUtil.match("/notice/**", () -> StpUtil.checkPermission("notice"));
              SaRouterUtil.match("/comment/**", () -> StpUtil.checkPermission("comment"));
  
              // 匹配 restful 风格路由 
              SaRouterUtil.match("/article/get/{id}", () -> StpUtil.checkPermission("article"));
  
              // 检查请求方式 
              SaRouterUtil.match("/notice/**", () -> {
                  if(request.getMethod().equals(HttpMethod.GET.toString())) {
                      StpUtil.checkPermission("notice");
                  }
              });
  
              // 在多账号模式下，可以使用任意StpUtil进行校验
              SaRouterUtil.match("/user/**", () -> StpUserUtil.checkLogin());
  
          })).addPathPatterns("/**");
      }
  }
  ```


### 框架配置

你可以**零配置启动框架**
但同时你也可以通过配置，定制性使用框架，`sa-token`支持多种方式配置框架信息

- `application.yml`配置

  ```yml
  spring: 
      # sa-token配置
      sa-token: 
          # token名称 (同时也是cookie名称)
          token-name: satoken
          # token有效期，单位s 默认30天, -1代表永不过期 
          timeout: 2592000
          # token临时有效期 (指定时间内无操作就视为token过期) 单位: 秒
          activity-timeout: -1
          # 是否允许同一账号并发登录 (为true时允许一起登录, 为false时新登录挤掉旧登录) 
          allow-concurrent-login: false
          # 在多人登录同一账号时，是否共用一个token (为true时所有登录共用一个token, 为false时每次登录新建一个token) 
          is-share: false
          # token风格
          token-style: uuid
  
  ```

- 通过代码配置

  ```java
  /**
   * sa-token代码方式进行配置
   */
  @Configuration
  public class SaTokenConfigure {
  
      // 获取配置Bean (以代码的方式配置sa-token, 此配置会覆盖yml中的配置)
      @Primary
      @Bean(name="SaTokenConfigure")
      public SaTokenConfig getSaTokenConfig() {
          SaTokenConfig config = new SaTokenConfig();
          config.setTokenName("satoken");             // token名称 (同时也是cookie名称)
          config.setTimeout(30 * 24 * 60 * 60);       // token有效期，单位s 默认30天
          config.setActivityTimeout(-1);              // token临时有效期 (指定时间内无操作就视为token过期) 单位: 秒
          config.setAllowConcurrentLogin(true);       // 是否允许同一账号并发登录 (为true时允许一起登录, 为false时新登录挤掉旧登录) 
          config.setIsShare(true);                    // 在多人登录同一账号时，是否共用一个token (为true时所有登录共用一个token, 为false时每次登录新建一个token) 
          config.setTokenStyle("uuid");               // token风格 
          return config;
      }
  
  }
  ```

- 配置项

	| 参数名称               | 类型    | 默认值  | 说明                                                         |
| ---------------------- | ------- | ------- | ------------------------------------------------------------ |
| tokenName              | String  | satoken | token名称 (同时也是cookie名称)                               |
| timeout                | long    | 2592000 | token有效期，单位/秒 默认30天，-1代表永久有效                |
| activityTimeout        | long    | -1      | token临时有效期 (指定时间内无操作就视为token过期) 单位: 秒, 默认-1 代表不限制 (例如可以设置为1800代表30分钟内无操作就过期) |
|                        |         |         | 是否允许同一账号并发登录 (为true时允许一起登录, 为false时新登录挤掉旧登录) |
| isShare                | Boolean | true    | 在多人登录同一账号时，是否共用一个token (为true时所有登录共用一个token, 为false时每次登录新建一个token) |
| isReadBody             | Boolean | true    | 是否尝试从请求体里读取token                                  |
| isReadHead             | Boolean | true    | 是否尝试从header里读取token                                  |
| isReadCookie           | Boolean | true    | 是否尝试从cookie里读取token                                  |
| tokenStyle             | String  | uuid    | token风格                                                    |
| dataRefreshPeriod      | int     | 30      | 默认dao层实现类中，每次清理过期数据间隔的时间 (单位: 秒) ，默认值30秒，设置为-1代表不启动定时清理 |
| tokenSessionCheckLogin | Boolean | true    | 获取token专属session时是否必须登录 (如果配置为true，会在每次获取token专属session时校验是否登录) |
| autoRenew              | Boolean | true    | 是否打开自动续签 (如果此值为true, 框架会在每次直接或间接调用getLoginId()时进行一次过期检查与续签操作) |
| tokenPrefix            | Boolean | true    | token前缀, 格式样例(satoken: Bearer xxxx-xxxx-xxxx-xxxx) [参考：token前缀](http://sa-token.dev33.cn/doc/index.html#/use/token-prefix) |
| isV                    | Boolean | true    | 是否在初始化配置时打印版本字符画                             |

## 深入

### 持久层扩展

Sa-token默认将会话数据保存在内存中，此模式读写速度最快，且避免了序列化与反序列化带来的性能消耗，但是此模式也有一些缺点，比如：重启后数据会丢失，无法在集群模式下共享数据

为此，sa-token将数据持久操作全部抽象到 `SaTokenDao` 接口中，保证大家对框架进行灵活扩展，比如我们可以将会话数据存储在 `Redis`、`Memcached`等专业的缓存中间件中，做到重启数据不丢失，而且保证分布式环境下多节点的会话一致性

除了框架内部对`SaTokenDao`提供的基于内存的默认实现，官方仓库还提供了以下扩展方案：

- 整合 Redis 

  引用以下其中一种

  ```xml
  <!-- sa-token整合redis (使用jdk默认序列化方式) -->
  <dependency>
      <groupId>cn.dev33</groupId>
      <artifactId>sa-token-dao-redis</artifactId>
      <version>1.19.0</version>
  </dependency>
  
  <!-- sa-token整合redis (使用jackson序列化方式) -->
  <dependency>
      <groupId>cn.dev33</groupId>
      <artifactId>sa-token-dao-redis-jackson</artifactId>
      <version>1.19.0</version>
  </dependency>
  
  ```

  > JDK       优点：兼容性好，缺点：Session序列化后基本不可读，对开发者来讲等同于乱码
  >
  > Jackson 优点：Session序列化后可读性强，可灵活手动修改，缺点：兼容性稍差

- 注意点

  无论使用哪种序列化方式，你都必须为项目提供一个Redis实例化方案，例如：

  ```xml
  <!-- 提供redis连接池 -->
  <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-pool2</artifactId>
  </dependency>
  ```

  添加redis 配置

  ```yml
  # 端口
  spring: 
      # redis配置 
      redis:
          # Redis数据库索引（默认为0）
          database: 1
          # Redis服务器地址
          host: 127.0.0.1
          # Redis服务器连接端口
          port: 6379
          # Redis服务器连接密码（默认为空）
          # password: 
          # 连接超时时间（毫秒）
          timeout: 1000ms
          lettuce:
              pool:
                  # 连接池最大连接数
                  max-active: 200
                  # 连接池最大阻塞等待时间（使用负值表示没有限制）
                  max-wait: -1ms
                  # 连接池中的最大空闲连接
                  max-idle: 10
                  # 连接池中的最小空闲连接
                  min-idle: 0
  ```

  > 集成`Redis`只需要引入对应的`pom依赖`即可，框架所有上层API保持不变

### 花式Token

- 内置

  ```java
  // 1. token-style=uuid    —— uuid风格 (默认风格)
  "623368f0-ae5e-4475-a53f-93e4225f16ae"
  
  // 2. token-style=simple-uuid    —— 同上，uuid风格, 只不过去掉了中划线
  "6fd4221395024b5f87edd34bc3258ee8"
  
  // 3. token-style=random-32    —— 随机32位字符串
  "qEjyPsEA1Bkc9dr8YP6okFr5umCZNR6W"
  
  // 4. token-style=random-64    —— 随机64位字符串
  "v4ueNLEpPwMtmOPMBtOOeIQsvP8z9gkMgIVibTUVjkrNrlfra5CGwQkViDjO8jcc"
  
  // 5. token-style=random-128    —— 随机128位字符串
  "nojYPmcEtrFEaN0Otpssa8I8jpk8FO53UcMZkCP9qyoHaDbKS6dxoRPky9c6QlftQ0pdzxRGXsKZmUSrPeZBOD6kJFfmfgiRyUmYWcj4WU4SSP2ilakWN1HYnIuX0Olj"
  
  // 6. token-style=tik    —— tik风格
  "gr_SwoIN0MC1ewxHX_vfCW3BothWDZMMtx__"
  ```

- 自定义Token生成

  重写`SaTokenAction`接口的`createToken`方法来实现

  新建文件`MySaTokenAction.java`，继承`SaTokenActionDefaultImpl`默认实现类, 并添加上注解`@Component`，保证此类被`springboot`扫描到

  ```java
  package com.pj.satoken;
  
  import org.springframework.stereotype.Component;
  import cn.dev33.satoken.action.SaTokenActionDefaultImpl;
  
  /**
   * 继承sa-token行为Bean默认实现, 重写部分逻辑 
   */
  @Component
  public class MySaTokenAction extends SaTokenActionDefaultImpl {
      // 重写token生成策略 
      @Override
      public String createToken(Object loginId, String loginKey) {
          return SaTokenInsideUtil.getRandomString(60);    // 随机60位字符串
      }
  }
  ```

- 雪花算法生成Token

  添加依赖

  ```xml
  <!-- Hutool 一个小而全的Java工具类库 -->
  <dependency>
      <groupId>cn.hutool</groupId>
      <artifactId>hutool-all</artifactId>
      <version>5.5.4</version>
  </dependency>
  ```

  `MySaTokenAction.java`，继承`SaTokenActionDefaultImpl`默认实现类, 并添加上注解`@Component`，保证此类被`springboot`扫描到

  ```java
  package com.pj.satoken;
  
  import org.springframework.stereotype.Component;
  import cn.dev33.satoken.action.SaTokenActionDefaultImpl;
  import cn.hutool.core.util.IdUtil;
  
  /**
   * 继承sa-token行为Bean默认实现, 重写部分逻辑 
   */
  @Component
  public class MySaTokenAction extends SaTokenActionDefaultImpl {
      // 重写token生成策略 
      @Override
      public String createToken(Object loginId, String loginKey) {
          return IdUtil.getSnowflake(1, 1).nextIdStr();    // 以雪花算法生成token 
      }
  }
  ```

### Token 前缀

- 配置

```yml
spring: 
    # sa-token配置
    sa-token: 
        # token前缀
        tokenPrefix: Bearer

```
- 接受格式

```json
{
    "satoken": "Bearer xxxx-xxxx-xxxx-xxxx"
}

```

> 1. `token前缀` 与 `token值` 之间必须有一个空格
> 2. 一旦配置了`token前缀`，则前端提交token时，必须带有前缀，否则会导致框架无法读取token
> 3. 由于`Cookie`中无法存储空格字符，也就意味配置token前缀后，`Cookie`鉴权方式将会失效，此时只能将token提交到`header`里进行传输

### 记住我模式

sa-token的登录授权，**默认就是`[记住我]`模式**，为了实现`[非记住我]`模式, 你需要在登录时如下设置：

```java
// 设置登录账号id为10001，第二个参数指定是否为[记住我]，当此值为false后，关闭浏览器后再次打开需要重新登录
StpUtil.setLoginId(10001, false);
```

**原理**

Cookie作为浏览器提供的默认会话跟踪机制，其生命周期有两种形式，分别是：

- 临时Cookie：有效期为本次会话，只要关闭浏览器窗口，Cookie就会消失
- 永久Cookie：有效期为一个具体的时间，在时间未到期之前，即使用户关闭了浏览器Cookie也不会消失

利用Cookie的此特性，我们便可以轻松实现 [记住我] 模式：

- 勾选[记住我]按钮时：调用`StpUtil.setLoginId(10001, true)`，在浏览器写入一个`永久Cookie`储存token，此时用户即使重启浏览器token依然有效
- 不勾选[记住我]按钮时：调用`StpUtil.setLoginId(10001, false)`，在浏览器写入一个`临时Cookie`储存token，此时用户在重启浏览器后token便会消失，导致会话失效

**前后端分离模式下实现**

此时机智的你😏很快发现一个问题，Cookie虽好，却无法在前后端分离环境下使用，那是不是代表上述方案在APP、小程序等环境中无效？

准确的讲，答案是肯定的，任何基于Cookie的认证方案在前后台分离环境下都会失效（原因在于这些客户端默认没有实现Cookie功能），不过好在，这些客户端一般都提供了替代方案， 唯一遗憾的是，此场景中token的生命周期需要我们在前端手动控制

以经典跨端框架 [uni-app](https://uniapp.dcloud.io/) 为例，我们可以使用如下方式达到同样的效果：

```js
// 使用本地存储保存token，达到 [永久Cookie] 的效果
uni.setStorageSync("satoken", "xxxx-xxxx-xxxx-xxxx-xxx");

// 使用globalData保存token，达到 [临时Cookie] 的效果
getApp().globalData.satoken = "xxxx-xxxx-xxxx-xxxx-xxx";

```

如果你决定在PC浏览器环境下进行前后台分离模式开发，那么更加简单：

```js
// 使用 localStorage 保存token，达到 [永久Cookie] 的效果
localStorage.setItem("satoken", "xxxx-xxxx-xxxx-xxxx-xxx");

// 使用 sessionStorage 保存token，达到 [临时Cookie] 的效果
sessionStorage.setItem("satoken", "xxxx-xxxx-xxxx-xxxx-xxx");
```

Remember me, it's too easy!

**登录指定Token有效期**

```java
// 示例1：
// 指定token有效期(单位: 秒)，如下所示token七天有效
StpUtil.setLoginId(10001, new SaLoginModel().setTimeout(60 * 60 * 24 * 7));

// ----------------------- 示例2：所有参数
// `SaLoginModel`为登录参数Model，其有诸多参数决定登录时的各种逻辑，例如：
StpUtil.setLoginId(10001, new SaLoginModel()
            .setDevice("PC")                // 此次登录的客户端设备标识, 用于[同端互斥登录]时指定此次登录的设备名称
            .setIsLastingCookie(true)        // 是否为持久Cookie（临时Cookie在浏览器关闭时会自动删除，持久Cookie在重新打开后依然存在）
            .setTimeout(60 * 60 * 24 * 7)    // 指定此次登录token的有效期, 单位:秒 （如未指定，自动取全局配置的timeout值）
            );
```

### 模拟他人

- 操作其他账号

  ```JAVA
  // 获取指定账号10001的`tokenValue`值 
  StpUtil.getTokenValueByLoginId(10001);
  
  // 将账号10001的会话注销登录（踢人下线）
  StpUtil.logoutByLoginId(10001);
  
  // 获取账号10001的Session对象, 如果session尚未创建, 则新建并返回
  StpUtil.getSessionByLoginId(10001);
  
  // 获取账号10001的Session对象, 如果session尚未创建, 则返回null 
  StpUtil.getSessionByLoginId(10001, false);
  
  // 获取账号10001是否含有指定角色标识 
  StpUtil.hasRole(10001, "super-admin");
  
  // 获取账号10001是否含有指定权限码
  StpUtil.hasPermission(10001, "user:add");
  ```

- 临时身份的切换

  ```java
  // 将当前会话[身份临时切换]为其它账号 
  StpUtil.switchTo(10044);
  
  // 此时再调用此方法会返回 10044 (我们临时切换到的账号id)
  StpUtil.getLoginId();
  
  // 结束 [身份临时切换]
  StpUtil.endSwitch();
  ```

  你还可以: 直接在一个代码段里方法内，临时切换身份为指定loginId (此方式无需手动调用`StpUtil.endSwitch()`关闭身份切换)

  ```java
  System.out.println("------- [身份临时切换]调用开始...");
  StpUtil.switchTo(10044, () -> {
      System.out.println("是否正在身份临时切换中: " + StpUtil.isSwitch()); 
      System.out.println("获取当前登录账号id: " + StpUtil.getLoginId());
  });
  System.out.println("------- [身份临时切换]调用结束...");
  ```


### 同端互斥登录

首先在配置文件中，将 `allowConcurrentLogin` 配置为false，然后调用登录等相关接口时声明设备标识即可：

- 指定设备登陆

  ```java
  // 指定`账号id`和`设备标识`进行登录
  StpUtil.setLoginId(10001, "PC");    
  ```

  >  调用此方法登录后，同设备的会被顶下线(不同设备不受影响)，再次访问系统时会抛出 `NotLoginException` 异常，场景值=`-4`

- 指定设备注销

    ```java
    // 指定`账号id`和`设备标识`进行强制注销 (踢人下线)
    StpUtil.logoutByLoginId(10001, "PC");    
    ```

    > 如果第二个参数填写null或不填，代表将这个账号id所有在线端踢下线，被踢出者再次访问系统时会抛出 `NotLoginException` 异常，场景值=`-5`

- 查询当前设备
    ```java
    // 返回当前token的登录设备
    StpUtil.getLoginDevice();   
    ```
    
- id反查token

    ```java
    // 获取指定loginId指定设备端的tokenValue 
    StpUtil.getTokenValueByLoginId(10001, "APP");
    ```

### 密码加密

- 摘要加密

  ```java
  // md5加密 
  SaSecureUtil.md5("123456");
  
  // sha1加密 
  SaSecureUtil.sha1("123456");
  
  // sha256加密 
  SaSecureUtil.sha256("123456");
  
  // md5加盐加密: md5(md5(str) + md5(salt)) 
  SaSecureUtil.md5BySalt("123456", "salt");
  
  ```

- 对称加密

  

- 非对称加密

- Base64编码和解码

  