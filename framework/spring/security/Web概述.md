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
    // 可以是Ant模式的Matcher，也可以是正则模式的Matcher
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

一般情况下 `RequestMatcher` 都会采用 `AntPathRequestMatcher` ，但是如果需要更加复杂的匹配规则的话可以使用 `RegexRequestMatcher`。在进行匹配的时候是只包含请求的 path 部分，不包含 path 中的参数和 QueryString。

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
    // 信任解析器，主要用于判断是否为匿名登录或remember-me登录
	private AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();
    // 经过包装的请求工厂（包含上述的所有内容），由该工程生成的请求可以执行身份认证相关的处理
    private HttpServletRequestFactory requestFactory;
}
```

这里的 `HttpServletRequestFactory` 采用的是 `HttpServlet3RequestFactory`，通过`HttpServlet3RequestFactory` 创建的 `SecurityContextHolderAwareRequestWrapper` 可以进行如下与身份认证相关的处理。

==这里的 `AuthenticationTrustResolver` 用处还不明了。==

### Servlet3SecurityContextHolderAwareRequestWrapper

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

==等需要的时候再了解，目前未使用JAAS认证。==

## RememberMeAuthenticationFilter

```java
public class RememberMeAuthenticationFilter extends GenericFilterBean implements
		ApplicationEventPublisherAware {
    // 身份认证成功后通过它发布InteractiveAuthenticationSuccessEvent事件
	private ApplicationEventPublisher eventPublisher;
    // 身份认证成功后调用它进行成功处理
	private AuthenticationSuccessHandler successHandler;
    // 对RememberMeServices解析出来的 Authentication 进行认证
	private AuthenticationManager authenticationManager;
    // 从请求中解析 RememberMeAuthenticationToken
	private RememberMeServices rememberMeServices;
    
    // 身份认证成功时调用，提供给子类进行扩展
    protected void onSuccessfulAuthentication(HttpServletRequest request,
			HttpServletResponse response, Authentication authResult) {
	}
    // 身份认证失败时调用，提供给子类进行扩展
    protected void onUnsuccessfulAuthentication(HttpServletRequest request,
			HttpServletResponse response, AuthenticationException failed) {
	}
}
```

当 SecurityContextHolder 中不存在 Authentication 时，该 Filter 会通过 RememberMeServices 从 Request 中解析出 Authentication ，然后通过 AuthenticationManager 进行身份认证。

### RememberMeServices

```java
public interface RememberMeServices {
    // 从请求中解析Authentication（一般为 RemeberMeAuthenticationToken）
    Authentication autoLogin(HttpServletRequest request, HttpServletResponse response);
    /**
	 * Called whenever an interactive authentication attempt was made, but the credentials
	 * supplied by the user were missing or otherwise invalid. Implementations should
	 * invalidate any and all remember-me tokens indicated in the HttpServletRequest.
	 */
    void loginFail(HttpServletRequest request, HttpServletResponse response);
    // 认证成功时根据请求中是否指定remember-me参数（QueryString或者FormData中）来决定是否需要向Response中写入remember-me token。
    void loginSuccess(HttpServletRequest request, HttpServletResponse response,
			Authentication successfulAuthentication);
}
```

### AbstractRememberMeServices

这个是 `RememberMeServices` 的基础实现，作为 `RememberMeServices` 实现类的基类。该基础实现默认从Cookie中解析 remember-me token，然后再调用子类实现获取 UserDetails。

```java
public abstract class AbstractRememberMeServices implements RememberMeServices, 	       InitializingBean, LogoutHandler {
    // remember-me token的cookie默认名称
    public static final String SPRING_SECURITY_REMEMBER_ME_COOKIE_KEY = "remember-me";
    // remember-me parameter默认名称，用于判断请求是否要求记录remember-me authentication，并将 remmeber-me token 写入Response
	public static final String DEFAULT_PARAMETER = "remember-me";
    // remember-me authentication的默认有效期（秒）
	public static final int TWO_WEEKS_S = 1209600;
    // remember-me token中不同部分之间的默认分隔符
	private static final String DELIMITER = ":";
    
    // remember-me token的cookie名称
    private String cookieName = SPRING_SECURITY_REMEMBER_ME_COOKIE_KEY;
    // remember-me token的cookie domain
	private String cookieDomain;
    // remember-me parameter名称
	private String parameter = DEFAULT_PARAMETER;
    // 是否对所有请求都开启 remember-me 模式，如果开启则对所有请求都会记录remember-me authentication，并将 remember-me token 写入Response
	private boolean alwaysRemember;
    // remember-me authentication的有效期（秒）
    private int tokenValiditySeconds = TWO_WEEKS_S;
    // 是否使用安全cookie
	private Boolean useSecureCookie = null;
    // 生成的 RememberMeAuthenticationToken 中的key值，需要与 RememberMeAuthenticationProvider 中的key值相对应。
    private String key;
    
    // 目前没有使用
    private UserDetailsService userDetailsService;
    // 用户账号和凭据有效性检查器
	private UserDetailsChecker userDetailsChecker = new AccountStatusUserDetailsChecker();
    // Authentication 中的details生成器（通过 getDetails()获取）
    private AuthenticationDetailsSource<HttpServletRequest, ?> authenticationDetailsSource = new WebAuthenticationDetailsSource();
    private GrantedAuthoritiesMapper authoritiesMapper = new NullAuthoritiesMapper();
    
    // 认证成功后的回调函数，供子类扩展用
    protected abstract void onLoginSuccess(HttpServletRequest request,
			HttpServletResponse response, Authentication successfulAuthentication);
    // 通过cookie中解析出的内容获取用户详细信息
    protected abstract UserDetails processAutoLoginCookie(String[] cookieTokens,
			HttpServletRequest request, HttpServletResponse response)
			throws RememberMeAuthenticationException, UsernameNotFoundException;
}
```

==这里需要着重说明的是`key`值，它必须与 `RememberMeAuthenticationProvider` 中的 `key` 值相对应，否则就算这里可以成功创建 `RememberMeAuthenticationToken` 也无法通过 `AuthenticationManager` 的身份认证。==

SpringSecurity默认提供了两种实现

- PersistentTokenBasedRememberMeServices：依赖于`PersistentTokenRepository`来对remember-me信息进行存储，生成的remember-me token格式为 *series:tokenValue*。
- TokenBasedRememberMeServices：直接将 remember-me信息全部存储在cookie中，生成的remember-me token 格式为 *username:expireTime:signatureValue*

## AnonymousAuthenticationFilter

```java
public class AnonymousAuthenticationFilter extends GenericFilterBean implements
		InitializingBean {
    // AnonymousAuthenticationToken.details生成器
    private AuthenticationDetailsSource<HttpServletRequest, ?> authenticationDetailsSource = new WebAuthenticationDetailsSource();
	// 生成 AnonymousAuthenticationToken 的key值，必须与 AnonymousAuthenticationProvider中的key值相对应
    private String key;
    // AnonymousAuthenticationToken 的统一凭据
	private Object principal;
    // AnonymousAuthenticationToken 的统一授权
	private List<GrantedAuthority> authorities;
}
```

当SecurityContextHolder 中的 Authentication 为null时，该Filter会生成一个 `AnonymousAuthenticationToken` 并设置到 SecurityContext 中。

## ExceptionTranslationFilter

```java
public class ExceptionTranslationFilter extends GenericFilterBean {
    private AccessDeniedHandler accessDeniedHandler = new AccessDeniedHandlerImpl();
    // 身份认证入口
	private AuthenticationEntryPoint authenticationEntryPoint;
    // Authentication信任解析器
	private AuthenticationTrustResolver authenticationTrustResolver = new AuthenticationTrustResolverImpl();
    // 异常分析器，通过它快速获取想要的异常
	private ThrowableAnalyzer throwableAnalyzer = new DefaultThrowableAnalyzer();
}
```

这里是它的最主要逻辑

- 如果是身份认证失败或者身份认证成功但是授权验证失败（且身份认证为匿名或remember-me认证），则调用 `AuthenticationEntryPoint` 进入登录入口。
- 否则直接调用 `AccessDeniedHandler` 进行授权失败处理。

```java
	private void handleSpringSecurityException(HttpServletRequest request,
			HttpServletResponse response, FilterChain chain, RuntimeException exception)
			throws IOException, ServletException {
        
		if (exception instanceof AuthenticationException) {
			sendStartAuthentication(request, response, chain,
					(AuthenticationException) exception);
		}
		else if (exception instanceof AccessDeniedException) {
			Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
			if (authenticationTrustResolver.isAnonymous(authentication) || authenticationTrustResolver.isRememberMe(authentication)) {
				sendStartAuthentication(
						request,
						response,
						chain,
						new InsufficientAuthenticationException(
							messages.getMessage(	"ExceptionTranslationFilter.insufficientAuthentication",
								"Full authentication is required to access this resource")));
			}
			else {
				accessDeniedHandler.handle(request, response,
						(AccessDeniedException) exception);
			}
		}
	}
```

