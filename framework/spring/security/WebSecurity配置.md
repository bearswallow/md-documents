可以通过 `@EnableWebSecurity` 来启用 WebSecurity 。

# WebSecurityConfiguration

这个是 WebSecurity 的自动化配置入口。

```java
// WebSecurity 的自动化配置，主要用于注册 springSecurityFilterChain。
@Configuration
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {
    
    // 注册命名为 springSecurityFilterChain 的 FilterChainProxy Bean
	@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)
    public Filter springSecurityFilterChain() throws Exception {
        // 判断是否已经对 WebSecurity 进行过配置。主要是判断 setFilterChainProxySecurityConfigurer 属性注入方法中是否已经对 WebSecurity 进行过配置
        boolean hasConfigurers = webSecurityConfigurers != null
            && !webSecurityConfigurers.isEmpty();
        
        // 如果没有，则创建一个默认的 WebSecurityConfigurer 对 WebSecurity 进行配置。
        if (!hasConfigurers) {
            WebSecurityConfigurerAdapter adapter = objectObjectPostProcessor
                .postProcess(new WebSecurityConfigurerAdapter() {
                });
            webSecurity.apply(adapter);
        }
        
        // 由 WebSecurity 构建 FilterChainProxy
        return webSecurity.build();
    }
    
    // WebSecurityConfiguration 类自身的依赖注入
    @Autowired(required = false)
	public void setFilterChainProxySecurityConfigurer(
			ObjectPostProcessor<Object> objectPostProcessor,
			@Value("#{@autowiredWebSecurityConfigurersIgnoreParents.getWebSecurityConfigurers()}") List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers)
			throws Exception {
        // 创建 WebSecurity，并进行 ObjectPostProcessor 处理
		webSecurity = objectPostProcessor.postProcess(new WebSecurity(objectPostProcessor));
		if (debugEnabled != null) {
			webSecurity.debug(debugEnabled);
		}

        // 将所有 WebSecurityConfigurer Bean 进行排序
		Collections.sort(webSecurityConfigurers, AnnotationAwareOrderComparator.INSTANCE);

        // 检查 WebSecurityConfigurer Bean 的Order是否重复，因为如果重复了就无法确定 WebSecurityConfigurer Bean 对 WebSecurity 的配置顺序
		Integer previousOrder = null;
		Object previousConfig = null;
		for (SecurityConfigurer<Filter, WebSecurity> config : webSecurityConfigurers) {
			Integer order = AnnotationAwareOrderComparator.lookupOrder(config);
			if (previousOrder != null && previousOrder.equals(order)) {
				throw new IllegalStateException(
						"@Order on WebSecurityConfigurers must be unique. Order of "
								+ order + " was already used on " + previousConfig + ", so it cannot be used on "
								+ config + " too.");
			}
			previousOrder = order;
			previousConfig = config;
		}
        
        // 按Bean的顺序，让每个 WebSecurityConfigurer 对 WebSecurity 进行配置
		for (SecurityConfigurer<Filter, WebSecurity> webSecurityConfigurer : webSecurityConfigurers) {
			webSecurity.apply(webSecurityConfigurer);
		}
        
        // 保存已经应用过配置的 WebSecurityConfigurer
		this.webSecurityConfigurers = webSecurityConfigurers;
	}
    
    // 提供获取Spring容器中注册的所有SecurityConfigurer类型的Bean的封装类的Bean，
    @Bean
	public static AutowiredWebSecurityConfigurersIgnoreParents autowiredWebSecurityConfigurersIgnoreParents(
			ConfigurableListableBeanFactory beanFactory) {
		return new AutowiredWebSecurityConfigurersIgnoreParents(beanFactory);
	}
    
}
```

==静态方法注解的@Bean会在所有@Configuration注解类实例化之前注册到Spring容器中，可以确保依赖于它的@Value能够得到正常解析==。所以这里的 `AutowiredWebSecurityConfigurersIgnoreParents` 也是采用静态方法定义的Bean，因为这样才能让 `WebSecurityConfiguration#setFilterChainProxySecurityConfigurer` 属性注入方法中的 @Value("#{@autowiredWebSecurityConfigurersIgnoreParents.getWebSecurityConfigurers()}") 能够得到正确的解析。

**AutowiredWebSecurityConfigurersIgnoreParents** 的实现也比较简单，就是从Spring容器中获取 `WebSecurityConfigurer` 类型的所有Bean。

```java
final class AutowiredWebSecurityConfigurersIgnoreParents {

	private final ConfigurableListableBeanFactory beanFactory;
    
	public List<SecurityConfigurer<Filter, WebSecurity>> getWebSecurityConfigurers() {
		List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers = new ArrayList<SecurityConfigurer<Filter, WebSecurity>>();
		Map<String, WebSecurityConfigurer> beansOfType = beanFactory
				.getBeansOfType(WebSecurityConfigurer.class);
		for (Entry<String, WebSecurityConfigurer> entry : beansOfType.entrySet()) {
			webSecurityConfigurers.add(entry.getValue());
		}
		return webSecurityConfigurers;
	}
}
```



# WebSecurityConfigurerAdapter

WebSecurityConfigurer 实现类的基类，已经定义了配置 WebSecurityConfigurer 的流程，也提供了默认的配置实现。并且引入了 `HttSecurity` ，把具体的配置工作都代理给它。

```java
@Order(100)
public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {
    
    // 初始化 WebSecurity 。初始化方法会在所有 Configurer 的 config 方法调用之前调用。
    public void init(final WebSecurity web) throws Exception {
		final HttpSecurity http = getHttp();
        // 添加默认的 SecurityFilterChain
		web.addSecurityFilterChainBuilder(http).postBuildAction(new Runnable() {
			public void run() {
                // 在 WebSecurity 的最后添加 FilterSecurityInterceptor ，完成授权认证。
				FilterSecurityInterceptor securityInterceptor = http
						.getSharedObject(FilterSecurityInterceptor.class);
				web.securityInterceptor(securityInterceptor);
			}
		});
	}
    
    // 对 WebSecurity 进行配置
    public void configure(WebSecurity web) throws Exception {
	}
    
    // 对默认的 SecurityFilterChain 进行配置
    protected void configure(HttpSecurity http) throws Exception {
		logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");

		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.formLogin().and()
			.httpBasic();
	}
    
}
```

# WebSecurity

`FilterChainProxy` 的构建类，提供了FluentAPI来对构建器的参数进行设置。

```java
public final class WebSecurity extends
		AbstractConfiguredSecurityBuilder<Filter, WebSecurity> implements
		SecurityBuilder<Filter>, ApplicationContextAware {
    // 不需要进行Security处理的请求匹配器列表，外部无法访问，只用于内部实现。
    private final List<RequestMatcher> ignoredRequests = new ArrayList<>();
    // 需要忽略的请求配置器，在 setApplicationContext 时初始化，提供给外部对需要忽略的请求匹配器进行设置，设置结果保存在 ignoredRequests 列表中。
    private IgnoredRequestConfigurer ignoredRequestRegistry;
    // 非法请求处理器
    private HttpFirewall httpFirewall;
    // 授权认证拦截器
    private FilterSecurityInterceptor filterSecurityInterceptor;
    // 是否启用调试模式，如果启用则在使用 WebSecurity 的应用程序的log日志中会打印 WebSecurity 注册的各种Filter，需要对日志进行设置 logging.level.org.springframework.security.web.FilterChainProxy=DEBUG
    private boolean debugEnabled;
    private WebInvocationPrivilegeEvaluator privilegeEvaluator;
	private DefaultWebSecurityExpressionHandler defaultWebSecurityExpressionHandler = new DefaultWebSecurityExpressionHandler();
	private SecurityExpressionHandler<FilterInvocation> expressionHandler = defaultWebSecurityExpressionHandler;
	// FilterChainProxy 构建完成后会调用该方法
	private Runnable postBuildAction = new Runnable() {
		public void run() {
		}
	};
    
    // 执行 FilterChainProxy 构建
    @Override
	protected Filter performBuild() throws Exception {
        // 至少包含一个 SecurityFilterChain 的构建器，即 FilterChainProxy 中至少包含一个 SecurityFilterChain ，要不然也无法对请求进行 Security 处理。
		Assert.state(
				!securityFilterChainBuilders.isEmpty(),
				() -> "At least one SecurityBuilder<? extends SecurityFilterChain> needs to be specified. "
						+ "Typically this done by adding a @Configuration that extends WebSecurityConfigurerAdapter. "
						+ "More advanced users can invoke "
						+ WebSecurity.class.getSimpleName()
						+ ".addSecurityFilterChainBuilder directly");
		int chainSize = ignoredRequests.size() + securityFilterChainBuilders.size();
		List<SecurityFilterChain> securityFilterChains = new ArrayList<>(
				chainSize);
        // 为每个需要忽略的请求匹配器添加一个 SecurityFilterChain，添加到列表中，在这些 SecurityFilterChain 中不包含任何 Filter，也就不会进行 Security 处理。
		for (RequestMatcher ignoredRequest : ignoredRequests) {
			securityFilterChains.add(new DefaultSecurityFilterChain(ignoredRequest));
		}
        // 为每个 SecurityFilterChainBuilder 构建 SecurityFilterChain，添加到列表中。
		for (SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder : securityFilterChainBuilders) {
			securityFilterChains.add(securityFilterChainBuilder.build());
		}
        
        // 用 SecurityFilterChain 列表创建 FilterChainProxy。
		FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);
		if (httpFirewall != null) {
			filterChainProxy.setFirewall(httpFirewall);
		}
		filterChainProxy.afterPropertiesSet();

		Filter result = filterChainProxy;
		if (debugEnabled) {
			logger.warn("\n\n"
					+ "********************************************************************\n"
					+ "**********        Security debugging is enabled.       *************\n"
					+ "**********    This may include sensitive information.  *************\n"
					+ "**********      Do not use in a production system!     *************\n"
					+ "********************************************************************\n\n");
			result = new DebugFilter(filterChainProxy);
		}
		postBuildAction.run();
		return result;
	}
    
}
```



# HttpSecurity

`SecurityFilterChain` 的构建类，提供了很多现成的方法来完成各种不同 Filter 的构建操作。

```java
public final class HttpSecurity extends AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity>
		implements SecurityBuilder<DefaultSecurityFilterChain>,
		HttpSecurityBuilder<HttpSecurity> {
           	
	// 需要添加到 SecurityFilterChain 中的Filter列表
    private List<Filter> filters = new ArrayList<>();
    // 当前 SecurityFilterChain 需要进行处理的请求匹配器
    private RequestMatcher requestMatcher = AnyRequestMatcher.INSTANCE;
    // 请求匹配器配置器，可以配置较为复杂的请求匹配器，配置结果存放在 requestMatcher 中。
    private final RequestMatcherConfigurer requestMatcherConfigurer;
    // Filter顺序比较器，里面维护了所有Filter的顺序
    private FilterComparator comparator = new FilterComparator();
    
    
    // 添加Filter，可以在指定的Filter的位置、之前、之后添加Filter，也可以直接以comparator中已有的Filter顺序添加该Filter
    public HttpSecurity addFilterAfter(Filter filter, Class<? extends Filter> afterFilter) {
		comparator.registerAfter(filter.getClass(), afterFilter);
		return addFilter(filter);
	}
    public HttpSecurity addFilterBefore(Filter filter,
			Class<? extends Filter> beforeFilter) {
		comparator.registerBefore(filter.getClass(), beforeFilter);
		return addFilter(filter);
	}
    public HttpSecurity addFilter(Filter filter) {
		Class<? extends Filter> filterClass = filter.getClass();
		if (!comparator.isRegistered(filterClass)) {
			throw new IllegalArgumentException(
					"The Filter class "
							+ filterClass.getName()
							+ " does not have a registered order and cannot be added without a specified order. Consider using addFilterBefore or addFilterAfter instead.");
		}
		this.filters.add(filter);
		return this;
	}
    public HttpSecurity addFilterAt(Filter filter, Class<? extends Filter> atFilter) {
		this.comparator.registerAt(filter.getClass(), atFilter);
		return addFilter(filter);
	}
    
            
    // 请求匹配器主要包含两种：AntPathRequestMatcher 和 RegexRequestMatcher。
    public HttpSecurity antMatcher(String antPattern) {
		return requestMatcher(new AntPathRequestMatcher(antPattern));
	}  
    public HttpSecurity regexMatcher(String pattern) {
		return requestMatcher(new RegexRequestMatcher(pattern, null));
	}
            
}
```

在 `HttpSecurity` 中定义了两个 `RequestMatcherConfigurer`

```java
public class RequestMatcherConfigurer
			extends AbstractRequestMatcherRegistry<RequestMatcherConfigurer> {
}

/**
 * An extension to RequestMatcherConfigurer that allows optionally configuring
 * the servlet path.
 */
public final class MvcMatchersRequestMatcherConfigurer extends RequestMatcherConfigurer {
}
```

