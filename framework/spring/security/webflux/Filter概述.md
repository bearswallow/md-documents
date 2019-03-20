# WebFilterChainProxy

这个类与mvc中的 `FilterChainProxy` 的作用是一样的，它是 Spring Security 在 WebFlux 中的入口。

```java
/**
 * WebFilter 链代理
 */
public class WebFilterChainProxy implements WebFilter {
    // 可以保存多个 SecurityWebFilterChain，匹配不同的请求
    private final List<SecurityWebFilterChain> filters;
}
```

其中，`WebFilter` 与 mvc 中的 Filter 类似，都是用于拦截请求的。

# SecurityWebFilterChain

这个类与 mvc 中的 `securityFilterChain` 的作用是一样的，用于配置不同的请求使用不同的Security WebFilter 链进行处理。

```java
public interface SecurityWebFilterChain {
    // 对WebFlux请求进行匹配
    Mono<Boolean> matches(ServerWebExchange exchange);
    // 对满足匹配的请求需要执行的 WebFilger 列表
    Flux<WebFilter> getWebFilters();
}
```

目前只有一个实现类 `MatcherSecurityWebFilterChain` 。

```java
public class MatcherSecurityWebFilterChain implements SecurityWebFilterChain {
    // WebFlux请求匹配器
	private final ServerWebExchangeMatcher matcher;
    // WebFilger 列表
	private final List<WebFilter> filters;
}
```

其中，`ServerWebExchangeMatcher` 有很多中实现。

- **NegatedServerWebExchangeMatcher**：反向匹配

  ```java
  public class NegatedServerWebExchangeMatcher implements ServerWebExchangeMatcher {
  	private final ServerWebExchangeMatcher matcher;
      
      @Override
  	public Mono<MatchResult> matches(ServerWebExchange exchange) {
  		return matcher.matches(exchange)
  			.flatMap(m -> m.isMatch() ? MatchResult.notMatch() : MatchResult.match());
  	}
  }
  ```

- **MediaTypeServerWebExchangeMatcher**：媒体类型匹配，通过请求头中的 `Accept` 媒体类型进行匹配

  ```java
  public class MediaTypeServerWebExchangeMatcher implements ServerWebExchangeMatcher {
      // 用于匹配的媒体类型集合
      private final Collection<MediaType> matchingMediaTypes;
      // 匹配时是否需要媒体类型完全相同，如果为false则在匹配时使用兼容匹配
  	private boolean useEquals;
      // 不用于匹配的媒体类型集合，请求头中的 Accept 媒体类型如果在这个集合中，则不用于匹配。
  	private Set<MediaType> ignoredMediaTypes = Collections.emptySet();
  }
  ```

- **PathPatternParserServerWebExchangeMatcher**：请求路径匹配，只有请求方法和请求路径都匹配时才算匹配成功

  ```java
  public final class PathPatternParserServerWebExchangeMatcher implements ServerWebExchangeMatcher {
      // 路径模式解析器，用于将字符串解析成 PathPattern
  	private static final PathPatternParser DEFAULT_PATTERN_PARSER = new PathPatternParser();
      // 路径模式，用于匹配请求路径
      private final PathPattern pattern;
      // 请求方法，用于匹配请求方法
  	private final HttpMethod method;
  }
  ```

- **OrServerWebExchangeMatcher**：组合或匹配器，多个匹配器中只要有一个匹配成功则算匹配成功

- **AndServerWebExchangeMatcher**：组合与匹配器，所有匹配器都匹配成功才算匹配成功

