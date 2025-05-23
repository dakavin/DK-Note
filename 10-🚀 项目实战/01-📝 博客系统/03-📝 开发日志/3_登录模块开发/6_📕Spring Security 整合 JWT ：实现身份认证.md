
## 0 流程分析

![image.png|800](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0acb8a75b807ec7a4b6a3c3bb58a8b8a.png)


- **第一步**：创建JwtTokenHelper工具类，==实现InitializingBean接口==
	- 先定义 token的==签发人==（从yml配置文件中获取）、签发token时使用的==密钥key==（基于Base64编码生成的）、解析token 的==jwt解析器==
	- 重写InitializingBean的方法，在JwtParser的所有属性被设置之后被调用，主要是==设置解析器的签发人、密钥key和最大容忍时差==
	- 根据username生成token的方法：通过签发人、发行时间、过期时间、密钥签名来配置token
	- 解析token的方法：调用解析器的parseClaimsJws方法，进行解析token，主要验证令牌的签名（key+issuer）是否异常、令牌格式是否正确、令牌是否过期

- **第二步**：创建密码编码配置类，使用BCrypt  来对密码进行加密（设置为暗文）

- **第三步**：创建用户详情服务UserDetailsServiceImpl实现类，实现UserDetailsService接口
	- 从数据源加载用户名和密码等角色信息
	- 根据上述信息，创建一个SpringSecurity所需的UserDetails对象（就是这个对象才是SS能够进行认证的对象）

- **第四步**：**（入口）** 创建认证过滤器`JwtAuthenticationFilter`，实现`AbstractAuthenticationProcessingFilter`类
	- 指定用户登录的url，都需要进行用户认证
	- 先将请求中传来的用户名和密码进行初步的验证（null 或者 空串）
		- 所以这里可以自定义一个异常类`UsernameOrPasswordNullException`
	- 将用户名和密码封装到`UsernamePasswordAuthenticationToken`，这样才能触发 `Spring Security` 的身份验证管理器==执行实际的身份验证过程==，然后返回身份验证结果

- **第五步**：认证结果的处理
	- 成功——RestAuthenticationSuccessHandler，实现AuthenticationSuccessHandler接口
		- 从认证管理中获取UserDetails对象，然后通过用户名使用jwt工具类生成token，并返回给前端
	- 失败——RestAuthenticationFailureHandler，实现AuthenticationFailureHandler接口
		- 判断异常的分类，从而返回不同的结果给前端
	- 延伸1——LoginRspVO，登录的响应参数类，只有token这个属性
	- 延伸2——ResultUtil，返参工具类，处理成功或失败的返参
	- 延伸3——添加登录异常和用户名或密码错误的业务异常码

- **第六步**：**集合配置**，JwtAuthenticationSecurityConfig，自定义JWT认证的配置
	- 设置JWT身份认证的过滤器
	- 设置JWT是否认证管理员
	- 设置登录认证对应的处理类
	- 。。。

- **第七步**：在admin模块中的WebSecurityConfig配置类，应用自定义的JWT认证配置
## 1 什么是 JWT？

JWT（JSON Web Token）是一种用于在不同应用之间安全传输信息的开放标准（RFC 7519）。它是一种基于 JSON 的轻量级令牌，由三部分组成：头部（Header）、载荷（Payload）和签名（Signature）。JWT 被广泛用于实现身份验证和授权，**特别适用于前后端分离的应用程序**。

令牌类似下面这一大长串：
```json
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJxdWFueGlhb2hhIiwiaXNzIjoicXVhbnhpYW9oYSIsImlhdCI6MTY5Mjk1OTY2MSwiZXhwIjoxNjkyOTYzMjYxfQ.wbqbn23C9vAe5sQZRCBzrIM4SiN1eNl55NIONmHoiPHPHSSu0QJGgPGUin80hA4XgMHEqN1Wm5KJlmKKucUyGQ
```

可以看到，由 `header.payload.signature` 三部分组成，你可以在此网站: [https://jwt.io/](https://jwt.io/) 上获得解析结果：

## 2 为什么要使用 JWT？

JWT 提供了一种在客户端和服务器之间传输安全信息的简单方法，具有以下优点：

- **无状态性（Stateless）**：JWT 本身包含了所有必要的信息，无需在服务器端存储会话信息，每个请求都可以独立验证。
- **灵活性**：JWT 可以存储任意格式的数据，使其成为传递用户信息、权限、角色等的理想选择。
- **安全性**：JWT 使用签名进行验证，确保信息在传输过程中不被篡改。
- **跨平台和跨语言**：由于 JWT 使用 JSON 格式，它在不同的编程语言和平台之间都可以轻松传递。

## 3 开始动手

### 3.1 添加 JWT 依赖

这里我们选择 Java JWT : JSON Web Token for Java and Android (简称 JJWT) 库。首先，在 `blog-springboot` 父模块中的 `pom.xml` 中声明版本号：
```xml
<properties>
    <jjwt.version>0.11.2</jjwt.version>  
</properties>

<!-- 统一依赖管理 -->  
<dependencyManagement>  
    <dependencies>
        <!--   JWT   -->  
        <dependency>  
            <groupId>io.jsonwebtoken</groupId>  
            <artifactId>jjwt-api</artifactId>  
            <version>${jjwt.version}</version>  
        </dependency>  
        <dependency>            <groupId>io.jsonwebtoken</groupId>  
            <artifactId>jjwt-impl</artifactId>  
            <version>${jjwt.version}</version>  
        </dependency>  
        <dependency>            <groupId>io.jsonwebtoken</groupId>  
            <artifactId>jjwt-jackson</artifactId>  
            <version>${jjwt.version}</version>  
        </dependency>  
    </dependencies>  
</dependencyManagement>
```

然后，在 `blog-module-jwt` 模块中的 `pom.xml` 文件中，引入该依赖，添加内容如下：
```xml
<dependencies>
    <!-- JWT -->  
    <dependency>  
        <groupId>io.jsonwebtoken</groupId>  
        <artifactId>jjwt-api</artifactId>  
    </dependency>  
    <dependency>        
	    <groupId>io.jsonwebtoken</groupId>  
        <artifactId>jjwt-impl</artifactId>  
    </dependency>  
    <dependency>        
	    <groupId>io.jsonwebtoken</groupId>  
        <artifactId>jjwt-jackson</artifactId>  
    </dependency>  
  
    <dependency>        
	    <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency>  
  
    <!-- 工具包 -->  
    <dependency>  
        <groupId>org.apache.commons</groupId>  
        <artifactId>commons-lang3</artifactId>  
    </dependency>  
  
    <!-- 通用模块 -->  
    <dependency>  
        <groupId>com.dakkk</groupId>  
        <artifactId>blog-module-common</artifactId>  
    </dependency>  
  
    <dependency>      
	    <groupId>com.fasterxml.jackson.core</groupId>  
        <artifactId>jackson-core</artifactId>  
    </dependency>  
</dependencies>
```

### 3.2 编写 JwtTokenHelper 工具类

接下来，我们在jwt这个模块下创建一个`utils`目录，然后封装一个 `JwtTokenHelper` 工具类，封装所有 `JWT` 相关的功能。

```java
@Component  
public class JwtTokenHelper implements InitializingBean {  
    /**  
     * 签发人  
     */  
    @Value("${jwt.issuer}")  
    private String issuer;  
  
    /**  
     * 存储签发 JWT 令牌时使用的密钥  
     */  
    private Key key;  
  
    /**  
     * 用于解析 JWT 令牌  
     */  
    private JwtParser jwtParser;  
  
    /**  
     * 接收一个 Base64 编码的密钥字符串  
     * 并将其解码为 Key 对象，用于签名和验证 JWT 令牌  
     */  
    @Value("${jwt.secret}")  
    public void setBase64Key(String base64Key){  
        key = Keys.hmacShaKeyFor(Base64.getDecoder().decode(base64Key));  
    }  
  
    /**  
     * 重写InitializingBean的方法，在JwtParser的所有属性被设置之后被调用，执行一些初始化操作  
     */  
    @Override  
    public void afterPropertiesSet() throws Exception {  
        // 指定令牌解析器的 签发者 和 密钥  
        // 考虑到不同服务器之间可能存在时钟偏移，setAllowedClockSkewSeconds  
        // 用于设置能够容忍的最大的时钟误差  
        jwtParser = Jwts.parserBuilder().requireIssuer(issuer)  
                .setSigningKey(key).setAllowedClockSkewSeconds(10)  
                .build();  
    }  
  
    /**  
     * 根据username生成一个 JWT 令牌(token)，该令牌使用 key 进行签名  
     * 令牌包含用户名和其他信息，如发行者和失效时间  
     */  
    public String generateToken(String username){  
        LocalDateTime now = LocalDateTime.now();  
        LocalDateTime expireTime = now.plusHours(1);  
  
        // 生成令牌  
        return Jwts.builder().setSubject(username)  
                //设置令牌的发行者  
                .setIssuer(issuer)  
                // 设置令牌的发行时间  
                .setIssuedAt(Date.from(now.atZone(ZoneId.systemDefault()).toInstant()))  
                // 设置令牌的过期时间  
                .setExpiration(Date.from(expireTime.atZone(ZoneId.systemDefault()).toInstant()))  
                // 通过密钥对令牌进行签名  
                .signWith(key)  
                // 完成令牌的构建过程  
                .compact();  
    }  
  
    /**  
     * 解析传入的JWT令牌（token），并验证签名和有效期  
     * 返回值：Jws->JWT令牌的轻量级封装，Claims->JWT令牌中包含的声明集合  
     */  
    public Jws<Claims> parseToken(String token){  
        try {  
            // 解析令牌，验证令牌签名，并提取令牌的声明  
            return jwtParser.parseClaimsJws(token);  
        // 签名异常、令牌格式错误、不支持的 JWT 异常或非法参数异常  
        } catch (SignatureException | MalformedJwtException | UnsupportedJwtException | IllegalArgumentException e){  
            throw new BadCredentialsException("Token 不可用", e);  
        // token过期异常  
        } catch (ExpiredJwtException e){  
            throw new CredentialsExpiredException("Token 失效", e);  
        }  
    }  
  
    /**  
     * 生成一个 Base64 的安全密钥字符串的方法  
     */  
    private static String generateBase64Key(){  
        // 生成安全密钥  
        Key secretKey = Keys.secretKeyFor(SignatureAlgorithm.HS512);  
        // 将密钥进行Base64编码  
        String base64Key = Base64.getEncoder().encodeToString(secretKey.getEncoded());  
  
        return base64Key;  
    }  
  
    /**  
     * 输出一个 Base64 的安全密钥  
     */  
    public static void main(String[] args) {  
        System.out.println("key: " + generateBase64Key());  
    }  
  
}
```

述代码中，`Token` 令牌的初始化工作在 `generateToken()` 方法中完成，主要是通过 `Jwts.builder` 返回的 `JwtBuilder` 来做的。令牌的解析工作交给了 `JwtParser` 类，在 `parseToken()` 方法中完成。

与之对应的，工具类中注入的一些参数，如 `jwt` 的签发人、秘钥，需要在 `applicaiton.yml` 中配置好：
```yml
jwt:
  # 签发人
  issuer: quanxiaoha
  # 秘钥（使用JwtTokenHelper中的main方法生成）
  secret: jElxcSUj38+Bnh73T68lNs0DfBSit6U3whQlcGO2XwnI+Bo3g4xsiCIPg8PV/L0fQMis08iupNwhe2PzYLB9Xg==
```

---

**如何生成安全的秘钥**？

小伙伴们可能会有疑问，配置文件中的这一串这么长的秘钥怎么来的？其实，小哈在 `JwtTokenHelper` 中，已经定义好了一个 `generateBase64Key()` 方法，它专门用于生成一个 Base64 的安全秘钥，执行 `main()` 方法即可，然后将生成好的秘钥配置到 `yml` 文件中。

### 3.3 PasswordEncoder 密码加密

在系统中，安全存储用户密码是至关重要的。使用明文存储密码容易受到攻击，相信小伙伴们都看过某些网站用户账户被黑，密码都是明文保存的新闻，因此使用密码加密技术来保护用户密码是必不可少的。

---

**密码加密的重要性**

- **安全性：** 存储加密后的密码可以防止明文密码泄漏，即使数据库被攻击也不会暴露用户的真实密码。
- **防御攻击：** 使用密码哈希算法可以防止常见的攻击，如彩虹表攻击。
- **隐私保护：** 用户的密码是敏感信息，应当采取措施来保护用户的隐私。

在 `blog-module-jwt` 模块中新建 `config` 包，并创建 `PasswordEncoderConfig` 配置类，代码如下：
```java
@Component  
public class PasswordEncoderConfig {  
    @Bean  
    PasswordEncoder passwordEncoder(){  
        // BCrypt 是一种安全且适合密码存储的哈希算法，它在进行哈希时会自动加入“盐”，增加密码的安全性。  
        return new BCryptPasswordEncoder();  
    }  
  
    /**  
     * 测试密码加盐  
     */  
    public static void main(String[] args) {  
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();  
        System.out.println(encoder.encode("123456"));  
    }  
}
```


> **PasswordEncoder 接口**
> 
> `PasswordEncoder` 接口是 Spring Security 提供的密码加密接口，它定义了密码加密和密码验证的方法。通过实现这个接口，您可以将密码加密为不可逆的哈希值，以及在验证密码时对比哈希值。

上述代码中，我们初始化了一个 `PasswordEncoder` 接口的具体实现类 `BCryptPasswordEncoder`。`BCryptPasswordEncoder` 是 Spring Security 提供的密码加密器的一种实现，使用 BCrypt 算法对密码进行加密。BCrypt 是一种安全且适合密码存储的哈希算法，它在进行哈希时会自动加入“盐”，增加密码的安全性。

### 3.4 实现 UserDetailsService：Spring Security 用户详情服务

**什么是 UserDetailsService？**

`UserDetailsService` 是 Spring Security 提供的接口，用于从应用程序的数据源（如数据库、LDAP、内存等）中加载用户信息。它是一个用于==将用户详情加载到 Spring Security 的中心机制==。`UserDetailsService` 主要负责两项工作：

1. **加载用户信息：** 从数据源中加载用户的用户名、密码和角色等信息。
2. **创建 UserDetails 对象：** 根据加载的用户信息，创建一个 Spring Security 所需的 `UserDetails` 对象，包含用户名、密码、角色和权限等。

---

**自定义实现类**

新建 `service` 包，并创建 `UserDetailServiceImpl` 实现类：
```java
@Service  
@Slf4j  
public class UserDetailsServiceImpl implements UserDetailsService {  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
        // todo 从数据库查询并获取用户信息  
  
        // 暂时先写死，用户为 zhangsan 密码使用加密过的Bcrypt密码  
        // authorities 用户指定角色，这里写死，将用户的角色设为 ADMIN 管理员  
        return User.withUsername("zhangsan")  
                .password("$2a$10$j6i/1HDGI0r98SEYfNZmW.6bban7mwWxl6ANGWu09xkhDTsrFgP8C")  
                .authorities("ADMIN")  
                .build();  
    }  
}
```

上述代码中，我们实现了 `UserDetailsService` 接口，并重写了 `loadUserByUsername()` 方法，该方法用于根据用户名加载用户信息的逻辑 ，正常需要从数据库中查询，这里我们先写死，继续开发后面的功能，后续再回过头来改造。

### 3.5 自定义认证过滤器

接下来，我们自定义一个用于认证的过滤器，新建 `/filter` 包，并创建 `JwtAuthenticationFilter` 过滤器，代码如下：
```java
public class JwtAuthenticationFilter extends AbstractAuthenticationProcessingFilter {  
    /**  
     * 指定用户登录的访问地址  
     */  
    public JwtAuthenticationFilter(){  
        super(new AntPathRequestMatcher("/login","POST"));  
    }  
  
    @Override  
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {  
        ObjectMapper mapper = new ObjectMapper();  
        // 解析提交的 JSON 数据  
        JsonNode jsonNode = mapper.readTree(request.getInputStream());  
        JsonNode usernameNode = jsonNode.get("username");  
        JsonNode passwordNode = jsonNode.get("password");  
  
        // 判断用户名、密码是否为空  
        if (Objects.isNull(usernameNode) || Objects.isNull(passwordNode)  
        || StringUtils.isBlank(usernameNode.textValue()) || StringUtils.isBlank(passwordNode.textValue())){  
            throw new UsernameOrPasswordNullException("用户名或密码不能为空");  
        }  
        String username = usernameNode.textValue();  
        String password = passwordNode.textValue();  
          
        // 将用户名、密码封装到Token中  
        UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken =   
                new UsernamePasswordAuthenticationToken(username,password);  
        return getAuthenticationManager().authenticate(usernamePasswordAuthenticationToken);  
    }  
}
```

此过滤器继承了 `AbstractAuthenticationProcessingFilter`，用于处理 JWT（JSON Web Token）的用户身份验证过程。在构造函数中，调用了父类 `AbstractAuthenticationProcessingFilter` 的构造函数，通过 `AntPathRequestMatcher` 指定了处理用户登录的访问地址。这意味着当请求路径匹配 `/login` 并且请求方法为 `POST` 时，该过滤器将被触发。

`attemptAuthentication()` 方法用于实现用户身份验证的具体逻辑。首先，我们解析了提交的 JSON 数据，并获取了用户名、密码，校验是否为空，若不为空，则将它们封装到 `UsernamePasswordAuthenticationToken` 中。最后，使用 `getAuthenticationManager().authenticate()` 来触发 Spring Security 的身份验证管理器==执行实际的身份验证过程==，然后返回身份验证结果。

---

**自定义用户名或密码不能为空异常**

上面过滤器代码中，有个动作是校验用户名、密码是否为空，为空则抛出 `UsernameOrPasswordNullException` 异常，此类是自定义的得来的。新建包 `/exception`, 在此包中创建该类：
```java
import org.springframework.security.core.AuthenticationException;

public class UsernameOrPasswordNullException extends AuthenticationException {  
    public UsernameOrPasswordNullException(String msg, Throwable cause) {  
        super(msg, cause);  
    }  
  
    public UsernameOrPasswordNullException(String msg) {  
        super(msg);  
    }  
}
```

注意，需继承自 `AuthenticationException`，只有该类型异常，才能被后续自定义的认证失败处理器捕获到。

### 3.6 自定义处理器

用户登录后，我们还需要处理其对应的结果，如登录成功，则返回 `Token` 令牌，登录失败，则返回对应的提示信息。在 Spring Security 中，`AuthenticationFailureHandler` 和 `AuthenticationSuccessHandler` 是用于处理身份验证失败和成功的接口。它们允许您在用户身份验证过程中自定义响应，以便更好地控制和定制用户体验。

---

**自定义认证成功处理器 RestAuthenticationSuccessHandler**

新建 `/handler` 包，并创建 `RestAuthenticationSuccessHandler` 类：
```java
@Component  
@Slf4j  
public class RestAuthenticaitonSuccessHandler implements AuthenticationSuccessHandler {  
    @Autowired  
    private JwtTokenHelper jwtTokenHelper;  
  
  
    @Override  
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {  
        // 从 authentication 对象中获取用户的 UserDetails 实例，这里获取用户的用户名  
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();  
  
        // 通过用户名生成 token        
        String username = userDetails.getUsername();  
        String token = jwtTokenHelper.generateToken(username);  
  
        // 返回 token        
        LoginRspVO loginRspVO = LoginRspVO.builder().token(token).build();  
  
        ResultUtil.ok(response, Response.success(loginRspVO));  
    }  
}
```
此类实现了 Spring Security 的 `AuthenticationSuccessHandler` 接口，用于处理身份验证成功后的逻辑。首先，从 authentication 对象中获取用户的 UserDetails 实例，这里是主要是获取用户的用户名，然后通过用户名生成 `Token` 令牌，最后返回数据。

---

**LoginRspVO**

此类是登录的响参类，`VO` (View Object) 表示和视图相关的实体类，`rsp` 是 `response` 的缩写，表示返参，对应的 `req` 是 `request` 的缩写，表示请求。小哈习惯上对 `VO` 类的命名规则是：_动作 + 请求标识/响应标识 + VO_，如 `LoginReqVO`、`LoginRspVO` 等，这样做的好处是，可以一眼看出此类的作用，方便后续维护。

`LoginRspVO` 实体类对应内容如下：
```java
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
@Builder  
public class LoginRspVO {  
    /**  
     * Token 值  
     */  
    private String token;  
}
```

---

**ResultUtil 返参工具类**

为了在过滤器中方便的返回 JSON 参数，我们需要封装一个工具类 `ResultUtil`， 放置在 `/utils` 包下，代码如下：
```java
public class ResultUtil {  
    /**  
     * 返回参数成功  
     *  
     * @param response  
     * @param result  
     * @throws IOException  
     */  
    public static void ok(HttpServletResponse response, Response<?> result) throws IOException {  
        response.setCharacterEncoding("UTF-8");  
        response.setStatus(HttpStatus.OK.value());  
        response.setContentType("application/json");  
        PrintWriter writer = response.getWriter();  
  
        ObjectMapper objectMapper = new ObjectMapper();  
        writer.write(objectMapper.writeValueAsString(result));  
        writer.flush();  
        writer.close();  
    }  
  
    /**  
     * 返回参数失败  
     * @param response  
     * @param result  
     * @throws IOException  
     */  
    public static void fail(HttpServletResponse response, Response<?> result) throws IOException {  
        response.setCharacterEncoding("UTF-8");  
        response.setStatus(HttpStatus.OK.value());  
        response.setContentType("application/json");  
        PrintWriter writer = response.getWriter();  
  
        ObjectMapper objectMapper = new ObjectMapper();  
        writer.write(objectMapper.writeValueAsString(result));  
        writer.flush();  
        writer.close();  
    }  
    /**  
     * 失败响参  
     * @param response  
     * @param status 可指定响应码，如 401 等  
     * @param result  
     * @throws IOException  
     */  
    public static void fail(HttpServletResponse response, Response<?> result,int status) throws IOException {  
        response.setCharacterEncoding("UTF-8");  
        response.setStatus(status);  
        response.setContentType("application/json");  
        PrintWriter writer = response.getWriter();  
  
        ObjectMapper objectMapper = new ObjectMapper();  
        writer.write(objectMapper.writeValueAsString(result));  
        writer.flush();  
        writer.close();  
    }  
}
```

---

**自定义认证失败处理器**

在 `/handler` 包下，创建 `RestAuthenticationFailureHandler` 认证失败处理器：
```java
@Component  
@Slf4j  
public class RestAuthenticaitonFailureHandler implements AuthenticationFailureHandler {  
    @Override  
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException {  
        log.warn("AuthenticationException: ",exception );  
        if (exception instanceof UsernameOrPasswordNullException){  
            // 用户名或密码为空  
            ResultUtil.fail(response, Response.fail(exception.getMessage()));  
        }else if(exception instanceof BadCredentialsException){  
            // 用户名或密码错误  
            ResultUtil.fail(response,Response.fail(ResponseErrorCodeEnum.USERNAME_OR_PWD_ERROR));  
        }  
  
        // 登录失败  
        ResultUtil.fail(response, Response.fail(ResponseErrorCodeEnum.LOGIN_FAIL));  
    }  
}
```

通过自定义了一个实现了 Spring Security 的 `AuthenticationFailureHandler` 接口类，用于在用户身份验证失败后执行一些逻辑。首先，我们打印了异常日志，方便后续定位问题，然后对异常的类型进行判断，通过 `ResultUtil` 工具类，返回不同的错误信息，如用户名或者密码为空、用户名或密码错误等，若未判断出异常是什么类型，则统一提示为 _登录失败_。

---

**ResponseCodeEnum**

编辑 `weblog-module-common` 模块中的 `ResponseCodeEnum` 枚举类，添加登录失败的响应码：
```java
LOGIN_FAIL("20000", "登录失败"),
USERNAME_OR_PWD_ERROR("20001", "用户名或密码错误"),
```

### 3.7 自定义 JWT 认证功能配置

完成了以上前置工作后，我们开始配置 `JWT` 认证相关的配置。在 `/config` 包下新建 `JwtAuthenticationSecurityConfig`, 代码如下：
```java
@Configuration  
public class JwtAuthenticationSecurityConfig extends  
        SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {  
    @Resource  
    private RestAuthenticationSuccessHandler restAuthenticaitonSuccessHandler;  
    @Resource  
    private RestAuthenticaitonFailureHandler restAuthenticaitonFailureHandler;  
    @Resource  
    private PasswordEncoder passwordEncoder;  
    @Resource  
    private UserDetailsService userDetailsService;  
  
    @Override  
    public void configure(HttpSecurity httpSecurity) throws Exception {  
        // 自定义的用于 JWT 身份验证的过滤器  
        JwtAuthenticationFilter filter = new JwtAuthenticationFilter();  
  
        // 设置过滤器的认证管理员，用于处理身份验证请求  
        filter.setAuthenticationManager(httpSecurity.getSharedObject(AuthenticationManager.class));  
  
        // 设置登录认证对应的处理类  
        filter.setAuthenticationSuccessHandler(restAuthenticaitonSuccessHandler);  
        filter.setAuthenticationFailureHandler(restAuthenticaitonFailureHandler);  
  
        // 直接使用 DaoAuthenticationProvider，它是 Spring Security 提供的 默认身份验证提供者 之一  
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();  
        // 将用户信息放入 身份提供者 中去  
        provider.setUserDetailsService(userDetailsService);  
        // 对用户密码设置加密算法  
        provider.setPasswordEncoder(passwordEncoder);  
        // 将身份验证提供者添加到 HTTP 安全策略中  
        httpSecurity.authenticationProvider(provider);  
        // 将这个jwt过滤器添加到 Spring Security 的过滤器链中  
        // 在 原生UsernamePasswordAuthenticationFilter过滤器 之前执行  
        httpSecurity.addFilterBefore(filter, UsernamePasswordAuthenticationFilter.class);  
    }  
}
```

上述代码是一个 Spring Security 配置类，用于配置 JWT（JSON Web Token）的身份验证机制。它继承了 Spring Security 的 `SecurityConfigurerAdapter` 类，用于在 Spring Security 配置中添加自定义的认证过滤器和提供者。通过重写 `configure()` 方法，我们将之前写好过滤器、认证成功、失败处理器，以及加密算法整合到了 `httpSecurity` 中。

### 3.8 应用 JWT 认证功能配置

接下来，我们编辑 `weblog-module-admin` 中的 Spring Security 配置 `WebSecurityConfig` 类，修改内容如下：
```java
@Configuration  
@EnableWebSecurity  
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {  
    @Autowired  
    private JwtAuthenticaitonSecurityConfig jwtAuthenticaitonSecurityConfig;  
    @Override  
    protected void configure(HttpSecurity http) throws Exception {  
        // 禁用 csrf        
        http.csrf().disable()  
                // 禁用表单登录  
                .formLogin().disable()  
                // 设置用户登录认证相关配置  
                .apply(jwtAuthenticaitonSecurityConfig)  
                .and()  
                .authorizeHttpRequests()  
                // 认证所有以 /admin 为前缀的 URL资源  
                .mvcMatchers("/admin/**").authenticated()  
                // 其他都需要放行，无需认证  
                .anyRequest().permitAll()  
                .and()  
                // 前后端分离，无需创建会话  
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);  
    }  
}
```

述代码中，在 `configure()` 方法中，首先禁用了 CSRF（Cross-Site Request Forgery）攻击防护。在前后端分离的情况下，通常不需要启用 CSRF 防护。同时，还禁用了表单登录，并应用了 `JWT` 相关的配置类 `JwtAuthenticationSecurityConfig`。最后，配置会话管理这块，将会话策略设置为无状态（`STATELESS`），适用于前后端分离的情况，无需创建会话。

**补充：前后端分离工程不需要csrf保护的原因**
![image.png|300](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/11b26ac4f5db32ab1776ece27420b793.png)
## 4 工程目录

上述功能代码搞定后，基本结构如下，小伙伴们可对照一下：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/ad426cd11d3ee6dc19c24b00dcdb730d.png)

## 5 测试一波登录接口

重启项目，访问 `http://localhost:8080/doc.html` 接口文档，因为 `Knife4j` 扫描的是 `Controller` 包，而 `/login` 接口定义在了过滤器 `JwtAuthenticationFilter` 中, 所以接口文档中并不会出现它。针对这个独特的接口，我们需要手动来调试它。

### 5.1 用户名、密码为空

手动改写接口请求路径为 `/login` 以及入参，测试一波用户名、密码为空的情况，可以看到提示信息正确：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/37801352150c73755536c7516b437326.png)

### 5.2 用户名、密码错误

再来测试一波用户名、密码错误的情况：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/507b069ce3e4a5d25783a80980a554cc.png)

**对应用户名错误，还不能校验，我们后续添加数据库操作后，就可以了**
### 5.3 用户名、密码正确

当用户名、密码正确的时候，接口应该返回 `Token` 令牌：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/c53f5cca79fc0661c387a2de2f823a67.png)

可以看到，正确返回了 `Token` 令牌，说明认证功能基本搞定了。

## 6 从数据库中查询用户信息

前面我们根据用户名查询用户信息这块，是代码中写死的。接下来，我们将其改造为从数据库中查询。首先，我们将 `t_user` 表中之前用于测试的记录删除干净：
```mysql
INSERT INTO `weblog`.`t_user` (`username`, `password`, `create_time`, `update_time`, `is_deleted`) VALUES ('dakkk', '$2a$10$n7RJ1q.RnXx5M3O6B0i0he04fZOPjIJpyWcKuicW1bFyFHWhlGose', now(), now(), 0);
```

然后, 编辑 `UserMapper` 接口，添加一个根据用户名查询信息的默认方法：
```java
public interface UserMapper extends BaseMapper<UserDO> {  
    default UserDO findByUsername(String username){  
        LambdaQueryWrapper<UserDO> lqw = new LambdaQueryWrapper<>();  
        lqw.eq(UserDO::getUsername,username);  
        return selectOne(lqw);  
    }  
}
```

最后，编辑 `UserDetailServiceImpl` 类，改为从数据库中查询：
```java
@Service  
@Slf4j  
public class UserDetailsServiceImpl implements UserDetailsService {  
    @Autowired  
    private UserMapper userMapper;  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
        // 从数据库中查询  
        UserDO userDO = userMapper.findByUsername(username);  
        // 判断用户是否存在  
        if (Objects.isNull(userDO)){  
            throw new UsernameNotFoundException("该用户不存在");  
        }  
  
        // authorities 用户指定角色，这里写死为 ADMIN 管理员  
        return User.withUsername(userDO.getUsername())  
                .password(userDO.getPassword())  
                .authorities("ADMIN")  
                .build();  
    }  
}
```

## 7 最后再测试一波

改走数据库中查询用户信息后，重启项目，最后我们再测试一下 `/login` 接口是否正常：
![image.png|200](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/04/0af22291a6ece60de55adbc8c5fcc1c5.png)

可以看到一切正常，走数据库验证用户名、密码也是 OK 的。

## 8 结语

Spring Security JWT 提供了一种安全且灵活的方式来实现身份验证和授权，适用于前后端分离的应用程序。通过使用 JWT，您可以实现无状态的身份验证机制，提高应用程序的安全性和可维护性。注意，本小节中，我们只完成了认证模块（用户登录），鉴权模块我们将在下小节中开发，敬请期待。