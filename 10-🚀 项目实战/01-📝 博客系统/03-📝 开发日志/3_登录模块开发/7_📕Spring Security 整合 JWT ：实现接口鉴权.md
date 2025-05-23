## 1 前言

![image.png|700](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/66cdc2a348563dbbee5f7d7ff35fa40d.png)


在现代的 Web 应用程序中，接口鉴权是一个重要的安全考虑因素。**通过接口鉴权，您可以确保只有经过身份验证和授权的用户能够访问特定的接口和资源。**

上小节中，我们通过 Spring Security + JWT 已经实现了用户认证功能，当用户名、密码输入正确时，返回 Token 令牌，否则提示错误信息。本小节中，我们将在后端程序中拿到此令牌，对接口进行鉴权，若令牌正确，则可正常访问接口资源，否则提示对应错误信息。
## 2 JWT 流程图

JWT 完整流程如下：
![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c8afbcda190fe0b0e9bc13a1e96389c2.png)

解释一下上面的流程图，主要包含以下几个步骤：
- 用户使用账号和密码发起 POST 请求，也就是上小节完成的登录认证接口；
- 服务器使用私钥创建一个 Token 令牌；
- 服务器返回这个 Token 令牌给浏览器；
- 浏览器将该 Token 令牌附带在请求头中，向服务器发送请求；
- 服务器验证该令牌；
- 若令牌正确，则返回响应的资源给浏览器。

其中，发起访问的请求头包含 `Token` 令牌，通常格式如下：
```http
Authorization: Bearer <token>
```

## 3 目前现状

目前的现状，虽然是有了登录认证的接口，但是登录完成后，当我们访问受保护的接口时，即使将 `Token` 令牌携带与请求一起发送，依然是无法请求成功。另外，提示信息如下图所示，非常不友好，和之前小节中定义好的统一返参格式也不一致。
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/597ba4b85323d0c04f9c65ec87ff0075.png)

接下来，我们要做的工作就是
- 让后端能够正常处理请求头中携带的 `Token` 令牌返回统一的响参格式
- 同时呢，也能针对不同错误，如令牌过期、令牌不可用的情况，返回对应的提示信息。

## 4 开始动手

### 4.1 新建 Token 校验过滤器

首先，我们在 `weblog-module-jwt` 模块下的 `/filter` 包下，创建 `TokenAuthenticationFilter` 过滤器，用于专门校验 `Token` 令牌，代码如下：
```java
@Slf4j  
public class TokenAuthenticationFilter extends OncePerRequestFilter {  
    @Resource  
    private JwtTokenHelper jwtTokenHelper;  
    @Resource  
    private UserDetailsService userDetailsService;  
    @Resource  
    private AuthenticationEntryPoint authenticationEntryPoint;  
  
    @Override  
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {  
        // 从请求头中获取 key 为 Authorization 的值  
        String header = request.getHeader("Authorization");  
  
        // 判断 value 值是否以 Bearer 开头  
        if (StringUtils.startsWith(header, "Bearer")) {  
            // 截取 token 令牌  
            String token = StringUtils.substring(header, 7);  
            log.info("Token:{}", token);  
  
            // 判空 Token            
            if (StringUtils.isNotBlank(token)) {  
                try {  
                    // 校验 token 是否可用，若解析异常，针对不同异常作出不同的响应参数  
                    jwtTokenHelper.validateToken(token);  
                } catch (SignatureException | MalformedJwtException | UnsupportedJwtException |  
                         IllegalArgumentException e) {  
                    // 抛出异常，统一让 AuthenticationEntryPoint 处理响应参数  
                    authenticationEntryPoint.commence(request, response, new AuthenticationServiceException("Token 不可用"));  
                    return;  
                } catch (ExpiredJwtException e) {  
                    authenticationEntryPoint.commence(request, response, new AuthenticationServiceException("Token 已失效"));  
                    return;  
                }  
  
                // 从token中解析出用户名  
                String username = jwtTokenHelper.getUsernameByToken(token);  
  
                if (StringUtils.isNotBlank(username)  
                        && Objects.isNull(SecurityContextHolder.getContext().getAuthentication())) {  
                    // 根据用户名获取用户的详细信息  
                    UserDetails userDetails = userDetailsService.loadUserByUsername(username);  
                    // 将用户信息存入  authentication，方便后续校验  
                    UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails,null,userDetails.getAuthorities());  
                    authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));  
                    // 将 authentication 存入 ThreadLocal，方便后续获取用户信息  
                    SecurityContextHolder.getContext().setAuthentication(authentication);  
                }  
            }  
        }  
        // 继续执行下一个过滤器  
        filterChain.doFilter(request,response);  
    }  
}
```

**⚠️⚠️⚠️ 将用户信息存入 Authorization的时候，一定要加上用户授权的角色**

上述代码中，我们自定义了一个 Spring Security 过滤器 `TokenAuthenticationFilter`，它继承了 `OncePerRequestFilter`，确保每个请求只被过滤一次。

在重写的 `doFilterInternal()` 方法中来定义过滤器处理逻辑
- 首先，从请求头中获取 `key` 为 `Authorization` 的值，判断是否以 Bearer 开头，若是，截取出 `Token`, 对其进行解析，并对可能抛出的异常做出不同的返参
- 最后，我们获取了用户详情信息，将用户信息存入 `authentication`，方便后续进行校验，同时将 `authentication` 存入 `ThreadLocal` 中，方便后面方便的获取用户信息。

#### 4.1.1 JwtTokenHelper 新增方法

上述过滤器中，需要在 `JwtTokenHelper` 工具类中添加两个方法：

- 校验 Token 是否可用；
- 解析 Token 获取用户名；

代码如下：
```java
/**  
 * 校验 token 是否可用  
 */  
public void validateToken(String token){  
    jwtParser.parseClaimsJws(token);  
}  
  
/**  
 * 解析 Token 获取用户名  
 */  
public String getUsernameByToken(String token){  
    try {  
        Claims claims = jwtParser.parseClaimsJws(token).getBody();  
        String username = claims.getSubject();  
        return username;  
    } catch (Exception e){  
        e.printStackTrace();  
    }  
    return null;  
}
```

### 4.2 添加处理器

#### 4.2.1 RestAuthenticationEntryPoint

在 `/handler` 包下，新增 `RestAuthenticationEntryPoint` 处理器，它专门用来==处理当用户未登录时，访问受保护的资源的情况==

代码如下：
```java
@Slf4j  
@Component  
public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {  
    @Override  
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {  
        log.warn("用户未登录，访问受保护的资源：",authException);  
        if(authException instanceof InsufficientAuthenticationException){  
            ResultUtil.fail(response, Response.fail(authException.getMessage()), Response.fail(ResponseErrorCodeEnum.UNAUTHORIZED));  
              
        }  
  
        ResultUtil.fail(response, Response.fail(authException.getMessage()), HttpStatus.UNAUTHORIZED.value());  
    }  
}
```

当请求相关受保护的接口时，请求头中未携带 `Token` 令牌，通过日志打印异常信息，你会发现对应的异常类型如下：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/716cc75acf554eed5639f751a2094164.png)

所以，我们在 `RestAuthenticationEntryPoint` 类中，首先处理的就是该类异常，返回的提示信息如下， `HTTP` 响应码为 `401`, 表示“未授权”（Unauthorized）。这个状态码表示客户端请求需要用户身份验证，但==提供的认证凭据无效，或者未提供认证凭据==：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/8e8caf2838d6cc348be9ec52d641ebed.png)

对应的，你需要在 `ResponseCodeEnum` 枚举类中添加 `UNAUTHORIZED` 枚举值，代码如下：

```java
UNAUTHORIZED("20002", "无访问权限，请先登录！"),
```

而 `ResultUtil.fail(response, HttpStatus.UNAUTHORIZED.value(), Response.fail(authException.getMessage()));` 这行代码，主要用于处理下图中，红框标注的，当校验 Token 异常时，捕获之，并主动调用 `commence()` 方法，返回不同的提示信息:
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1d149071b75304e70c8942f106c0f11b.png)

---

**Token 失效返参**

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/6c0c04d1ccfe61dc472030e29e774c7a.png)

---

**Token 不可用返参**

![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/bcedd64dc50911302160c721525d2833.png)

可以看到，现在针对不同的错误场景，返参对应的提示信息都非常友好。

#### 4.2.2 RestAccessDeniedHandler

然后，在 `/handler` 包下，另外新增 `RestAccessDeniedHandler` 处理器，它主要用于==处理当用户登录成功时，访问受保护的资源，但是权限不够==的情况：
```java
@Slf4j  
@Component  
public class RestAccessDeniedHandler  implements AccessDeniedHandler {  
  
    @Override  
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {  
        log.warn("登录成功访问受保护的资源，权限不够：{}",accessDeniedException);  
        // todo 预留，后面引入角色的时候用到  
    }  
}
```

### 4.3 修改 Spring Security 配置

完成以上 `Token` 校验过滤器相关的开发后，接下来，需要将它们添加到 `Spring Security` 中，编辑 `WebSecurityConfig` 配置类，修改内容如下：
```java
@Configuration  
@EnableWebSecurity  
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {  
    @Resource  
    private JwtAuthenticationSecurityConfig jwtAuthenticaitonSecurityConfig;  
    @Resource  
    private RestAuthenticationEntryPoint authEntryPoint;  
    @Resource  
    private RestAccessDeniedHandler deniedHandler;  
    @Override  
    protected void configure(HttpSecurity http) throws Exception {  
        // 禁用 csrf        http.csrf().disable()  
                // 禁用表单登录,使用 JWT 登录不需要表单登录  
                .formLogin().disable()  
                // 应用jwt模块中自定义的用户认证配置  
                .apply(jwtAuthenticaitonSecurityConfig)  
                .and()  
                // 后续是配置 HTTP 请求的认证规则  
                .authorizeHttpRequests()  
                // 认证所有以 /admin 为前缀的 URL资源  
                .mvcMatchers("/admin/**").authenticated()  
                // 其他都需要放行，无需认证  
                .anyRequest().permitAll()  
                .and()  
                // 处理用户未登录访问受保护的资源的情况  
                .httpBasic().authenticationEntryPoint(authEntryPoint)  
                .and()  
                // 处理登录成功后访问受保护的资源，但是权限不够的情况  
                .exceptionHandling().accessDeniedHandler(deniedHandler)  
                .and()  
                // 前后端分离，设置会话管理策略为无状态的,即无需创建会话  
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)  
                .and()  
                // 将 Token 校验过滤器添加到用户认证过滤器之前  
                .addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);  
  
    }  
  
    /**  
     * token 校验过滤器  
     */  
    @Bean  
    public TokenAuthenticationFilter tokenAuthenticationFilter(){  
        return new TokenAuthenticationFilter();  
    }  
}
```

上述代码中，我们注入了前面写好的相关类，==如过滤器、处理器==等，主要改动代码如下：
- `.httpBasic().authenticationEntryPoint(authEntryPoint)` : 处理用户未登录访问受保护的资源的情况；
- `.exceptionHandling().accessDeniedHandler(deniedHandler)` : 处理登录成功后访问受保护的资源，但是权限不够的情况;
- `.addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)` : 将 Token 校验过滤器添加到用户认证过滤器之前;

## 5 测试看看

### 5.1 先登录

我们先调用登录接口，传入正确的用户名、密码，获取到一个新的 `Token` 令牌：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/d5b7d1c692a5a3a67f69e2101f584294.png)

### 5.2 设置全局请求头参数

Knife4j 允许我们设置全局请求头参数，将上面登录接口返回的 `Token` 令牌复制，然后操作如下：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/1592a33bd8c00fd5e629d0f8bb8bceaa.png)

_添加参数_，格式按照统一约定好的，参数名为 `Authorization` , 参数值的格式为 `Bearer 令牌`， 注意，需要以 `Bearer` 为前缀，中间空一格，然后点击确定按钮即可。
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/78dfeb0af6b01ec3acd12160d1006316.png)

刷新页面，在后面的请求接口中，就会自动带上这个 `Token` 请求头啦~ 有了 `Token` 令牌，我们就可以正常的请求被保护的接口了。

## 6 相关变量提取到 application.yml 中

鉴权功能正常开发完毕后，你会发现已经开发好的代码中，还有一些写死的变量，如请求头的 `key`, 令牌的失效时间等，其实都是可以提取到 `applicaiton.yml` 配置文件中，方便后续统一维护的：
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/e6ebd3d6edb3c62919fe3b9bc74b936e.png)

编辑 `application.yml` 文件, 添加如下配置：
```yml
jwt:
  # token 过期时间（单位：分钟） 24*60
  tokenExpireTime: 1440
  # token 请求头中的 key 值
  tokenHeaderKey: Authorization
  # token 请求头中的 value 值前缀
  tokenPrefix: Bearer
```

编辑 `JwtTokenHelper` 类，修改内容如下：
```java
public class JwtTokenHelper implements InitializingBean {
	// 省略...

    /**
     * Token 失效时间（分钟）
     */
    @Value("${jwt.tokenExpireTime}")
    private Long tokenExpireTime;

    public String generateToken(String username) {
        LocalDateTime now = LocalDateTime.now();
        // 设置 Token 失效时间
        LocalDateTime expireTime = now.plusMinutes(tokenExpireTime);

        return Jwts.builder()
                .setSubject(username)
                .setIssuer(issuer)
                .setIssuedAt(Date.from(now.atZone(ZoneId.systemDefault()).toInstant()))
                .setExpiration(Date.from(expireTime.atZone(ZoneId.systemDefault()).toInstant()))
                .signWith(key)
                .compact();
    }
	// 省略...
}
```

编辑 `TokenAuthenticationFilter` , 修改内容如下：
```java
@Slf4j
public class TokenAuthenticationFilter extends OncePerRequestFilter {

    @Value("${jwt.tokenPrefix}")
    private String tokenPrefix;
    
    @Value("${jwt.tokenHeaderKey}")
    private String tokenHeaderKey;
    
    // 省略...

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 从请求头中获取 key 为 Authorization 的值
        String header = request.getHeader(tokenHeaderKey);

        // 判断 value 值是否以 Bearer 开头
        if (StringUtils.startsWith(header, tokenPrefix)) 
        	// 省略

}
```

到这里，这些写死的变量值优化工作就搞定了。

## 7 结语

整合 Spring Security 与 JWT 不仅增强了应用程序的安全性，还提供了更加灵活的访问授权机制。通过这小节内容，我们已经在 `weblog` 项目中实现了身份验证和鉴权的功能，后台接口已经完成，下面小节中，我们将重新回到前端，完善之前的登录静态页面，当用户点击登录按钮时，请求真实的后台登录接口，并跳转 `Admin` 管理后台。