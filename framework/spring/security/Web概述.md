Spring Security 是纯粹依赖于标准的 servlet filter，不依赖于任何的 servlets 和 相关的实现框架（如：SpringMVC），就是处理 `HttpServletRequest` 和 `HttpServletResponse` 。

Spring Security 维护了一个 filter chain，请求会按 filter chain 中的顺序逐个执行 filter，所以 filter 在 filter chain 中的顺序是很重要的。Spring Security 提供了一个预先定义好的 filter chain，没有特殊需求可以不用关注 filter chain 的构建。

# FilterChainProxy

SpringSecurity 与 Web 整合的方式就是在 `web.xml` 中定义一个 `DelegatingFilterProxy`，并且该 `DelegatingFilterProxy`  的真实 Servlet Filter Bean 就是 `FilterChainProxy`，默认的 bean 名称为 `filterChainProxy`。

```xml
<filter>
    <filter-name>filterChainProxy</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
    <filter-name>filterChainProxy</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

当然，将SpringSecurity需要的每个Servlet Filter定义到 `web.xml` 中也是可以的，但是这样做会迅速让 `web.xml` 膨胀，也会导致频繁的修改 `web.xml` 文件。

```java
public class FilterChainProxy extends GenericFilterBean {
    // 经过处理的 ServletRequest 请求中会被设置一个属性，表示该请求是经过 FilterChainProxy 处理的请求，这个属性名就是这里设置的。
    private final static String FILTER_APPLIED = FilterChainProxy.class.getName().concat(
			".APPLIED");
	
    // SecurityFilterChain列表，针对不同的url可以定义不同的SecurityFilterChain。会根据列表顺序遍历所有的SecurityFilterChain，对当前请求的url进行匹配，会采用第一个匹配的SecurityFilterChain进行处理，所以需要将url限定最严格的SecurityFilterChain放在最前面。
	private List<SecurityFilterChain> filterChains;
	
    // SecurityFilterChain校验器，需要检查SecurityFilterChain列表是否合法
	private FilterChainValidator filterChainValidator = new NullFilterChainValidator();
	// 在调用SecurityFilterChain执行Security处理之前对非法的请求进行处理，直接拒绝非法请求。默认使用 StrictHttpFirewall。
	private HttpFirewall firewall = new StrictHttpFirewall();
}
```

**SecurityFilterChain** 的定义如下：

```java
public final class DefaultSecurityFilterChain implements SecurityFilterChain {
	private final RequestMatcher requestMatcher;
	private final List<Filter> filters;
    
    // 实现SecurityFilterChain接口的方法
    public List<Filter> getFilters() {
		return filters;
	}

    // 实现SecurityFilterChain接口的方法
	public boolean matches(HttpServletRequest request) {
		return requestMatcher.matches(request);
	}
}
```

# Filter Ordering

- `ChannelProcessingFilter`, because it might need to redirect to a different protocol
- `SecurityContextPersistenceFilter`, so a `SecurityContext` can be set up in the `SecurityContextHolder` at the beginning of a web request, and any changes to the `SecurityContext` can be copied to the `HttpSession` when the web request ends (ready for use with the next web request)
- `ConcurrentSessionFilter`, because it uses the `SecurityContextHolder` functionality and needs to update the `SessionRegistry` to reflect ongoing requests from the principal
- Authentication processing mechanisms - `UsernamePasswordAuthenticationFilter`, `CasAuthenticationFilter`, `BasicAuthenticationFilter` etc - so that the `SecurityContextHolder` can be modified to contain a valid `Authentication` request token
- The `SecurityContextHolderAwareRequestFilter`, if you are using it to install a Spring Security aware `HttpServletRequestWrapper` into your servlet container
- The `JaasApiIntegrationFilter`, if a `JaasAuthenticationToken` is in the `SecurityContextHolder` this will process the `FilterChain` as the `Subject` in the `JaasAuthenticationToken`
- `RememberMeAuthenticationFilter`, so that if no earlier authentication processing mechanism updated the `SecurityContextHolder`, and the request presents a cookie that enables remember-me services to take place, a suitable remembered `Authentication` object will be put there
- `AnonymousAuthenticationFilter`, so that if no earlier authentication processing mechanism updated the `SecurityContextHolder`, an anonymous `Authentication` object will be put there
- `ExceptionTranslationFilter`, to catch any Spring Security exceptions so that either an HTTP error response can be returned or an appropriate `AuthenticationEntryPoint` can be launched
- `FilterSecurityInterceptor`, to protect web URIs and raise exceptions when access is denied

## SecurityContextPersistenceFilter

```java
// SecurityContext持久化Filter
public class SecurityContextPersistenceFilter extends GenericFilterBean {
    // 经过该Filter处理后会在Request中设置相应的Attribute，这是Attribute的名称。
    static final String FILTER_APPLIED = "__spring_security_scpf_applied";
    // SecurityContext存储接口，执行真正的持久化操作
    private SecurityContextRepository repo;
    // 是否为不包含Session的Request生成新的Session，默认不会
    private boolean forceEagerSessionCreation = false;
}
```

如果使用 `HttpSessionSecurityContextRepository` 实现，则 SecurityContext 会整个存放到 Response 的Session中。

## ConcurrentSessionFilter

```java
// Session处理Filter，检测当前请求对应的Session是否过期，如果实现则进行登出处理并发布相应事件，如果未过期则刷新其最后更新时间，防止Session过期。
public class ConcurrentSessionFilter extends GenericFilterBean {
    // Session存储接口
    private final SessionRegistry sessionRegistry;
    // 登出处理器，在检测到Session过期后需要调用该处理器进行登出处理。
    private LogoutHandler handlers = new CompositeLogoutHandler(new SecurityContextLogoutHandler());
    // Session过期的处理策略，通过它发布SessionInformationExpiredEvent事件
	private SessionInformationExpiredStrategy sessionInformationExpiredStrategy;
}
```

## SecurityContextHolderAwareRequestFilter

```java
public class SecurityContextHolderAwareRequestFilter extends GenericFilterBean {
    // 角色前缀
    private String rolePrefix = "ROLE_";
	// 身份认证不通过时的处理入口
	private AuthenticationEntryPoint authenticationEntryPoint;
	// 认证管理器，当前请求用户身份认证的实际执行者
	private AuthenticationManager authenticationManager;
	// 登出处理器列表
	private List<LogoutHandler> logoutHandlers;
    // 信任解析器，主要用于判断是否为匿名登录或已经记录登录
	private AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();
    // 经过包装的请求工厂（包含上述的所有内容），由该工程生成的请求可以执行身份认证相关的处理
    private HttpServletRequestFactory requestFactory;
}
```

这里的 `HttpServletRequestFactory` 采用的是 `HttpServlet3RequestFactory`，通过`HttpServlet3RequestFactory` 创建的 `SecurityContextHolderAwareRequestWrapper` 可以进行如下与身份认证相关的处理。

```java
private class Servlet3SecurityContextHolderAwareRequestWrapper
			extends SecurityContextHolderAwareRequestWrapper {
    // 判断当前请求是否通过身份认证，如果未通过认证则由 AuthenticationEntryPoint 执行登录
    @Override
    public boolean authenticate(HttpServletResponse response) {
        ...
    }
    
    // 通过账号密码进行用户身份认证操作，由 AuthenticationManager 完成认证操作
    @Override
    public void login(String username, String password) throws ServletException {
        ...
    }
    
    // 通过LogoutHandler执行登录操作
    @Override
    public void logout() throws ServletException {
        ...
    }
}
```

但是这里的login成功后只是会将通过认证的 `AuthenticationToken` 保存到 `SecurityContext` 中，并没有 调用 `AuthenticationSuccessHandler` 进行认证成功的执行。

## JaasApiIntegrationFilter

等需要的时候再了解，目前未使用JAAS认证。

## RememberMeAuthenticationFilter

