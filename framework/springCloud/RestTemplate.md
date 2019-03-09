# RestTemplate重要字段

```java
public class RestTemplate {
    
	private final List<HttpMessageConverter<?>> messageConverters = new ArrayList<>();
	private ResponseErrorHandler errorHandler = new DefaultResponseErrorHandler();
	private UriTemplateHandler uriTemplateHandler;
	private final ResponseExtractor<HttpHeaders> headersExtractor = new HeadersExtractor();
    
    // 从InterceptingHttpAccessor继承而来的
    private final List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();
	@Nullable
	private volatile ClientHttpRequestFactory interceptingRequestFactory;

}
```

上面的重要字段也意味着它可以被定制的地方。

## HttpMessageConverter

这个是很重要的，用于Http请求和响应内容序列化和反序列化处理的。

```java
	public RestTemplate() {
		this.messageConverters.add(new ByteArrayHttpMessageConverter());
		this.messageConverters.add(new StringHttpMessageConverter());
		this.messageConverters.add(new ResourceHttpMessageConverter(false));
		try {
			this.messageConverters.add(new SourceHttpMessageConverter<>());
		}
		catch (Error err) {
			// Ignore when no TransformerFactory implementation is available
		}
		this.messageConverters.add(new AllEncompassingFormHttpMessageConverter());

		if (romePresent) {
			this.messageConverters.add(new AtomFeedHttpMessageConverter());
			this.messageConverters.add(new RssChannelHttpMessageConverter());
		}

		if (jackson2XmlPresent) {
			this.messageConverters.add(new MappingJackson2XmlHttpMessageConverter());
		}
		else if (jaxb2Present) {
			this.messageConverters.add(new Jaxb2RootElementHttpMessageConverter());
		}

		if (jackson2Present) {
			this.messageConverters.add(new MappingJackson2HttpMessageConverter());
		}
		else if (gsonPresent) {
			this.messageConverters.add(new GsonHttpMessageConverter());
		}
		else if (jsonbPresent) {
			this.messageConverters.add(new JsonbHttpMessageConverter());
		}

		if (jackson2SmilePresent) {
			this.messageConverters.add(new MappingJackson2SmileHttpMessageConverter());
		}
		if (jackson2CborPresent) {
			this.messageConverters.add(new MappingJackson2CborHttpMessageConverter());
		}

		this.uriTemplateHandler = initUriTemplateHandler();
	}
```

构造函数已经为其添加了很多默认实现。主要依靠内部类`HttpEntityRequestCallback` 和 `ResponseEntityResponseExtractor` 完成。

```java
// 处理序列化
private class HttpEntityRequestCallback extends AcceptHeaderRequestCallback
// 处理反序列化
private class ResponseEntityResponseExtractor<T> implements ResponseExtractor<ResponseEntity<T>>
```

# RestTemplate 配置

通过使用 `RestTemplateBuilder` 进行构建，主要通过 `RestTemplateAutoConfiguration` 进行自动化配置，而该自动化配置还依赖于其它第三方 `HttpMessageConverter` 的自动化配置：

```java
@Configuration
@AutoConfigureAfter(HttpMessageConvertersAutoConfiguration.class)
@ConditionalOnClass(RestTemplate.class)
public class RestTemplateAutoConfiguration {
	private final ObjectProvider<HttpMessageConverters> messageConverters;
	private final ObjectProvider<RestTemplateCustomizer> restTemplateCustomizers;
}
```

