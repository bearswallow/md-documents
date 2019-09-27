# 注册 `@EventListener` 方法处理器

完整类名：`org.springframework.context.annotation.AnnotationConfigUtils`。

```java
public abstract class AnnotationConfigUtils {
    
    /**
	 * The bean name of the internally managed @EventListener annotation processor.
	 */
	public static final String EVENT_LISTENER_PROCESSOR_BEAN_NAME =
			"org.springframework.context.event.internalEventListenerProcessor";

	/**
	 * The bean name of the internally managed EventListenerFactory.
	 */
	public static final String EVENT_LISTENER_FACTORY_BEAN_NAME =
			"org.springframework.context.event.internalEventListenerFactory";
    
    public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, @Nullable Object source) {
        Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);
        // ...省略与event无关的代码
        
        // 注册默认使用的@EventListener方法处理器EventListenerMethodProcessor
        if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
		}

        // 注册默认使用的EventListenerFactory
        // DefaultEventListenerFactory，作为ApplicationListener的工厂
		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
		}

		return beanDefs;
    }
    
    private static BeanDefinitionHolder registerPostProcessor(
			BeanDefinitionRegistry registry, RootBeanDefinition definition, String beanName) {

		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(beanName, definition);
		return new BeanDefinitionHolder(definition, beanName);
	}
    
}
```

# `@EventListener` 方法处理器

完整类名：`org.springframework.context.event.EventListenerMethodProcessor`。

`EventListenerMethodProcessor` 会将所有 `@Component`(及其派生注解)所注解的Bean中具有 `@EventListener` (及其派生注解)所注解的方法注册到 `ApplicationContext` 中。

```java
public class EventListenerMethodProcessor
		implements SmartInitializingSingleton, ApplicationContextAware, BeanFactoryPostProcessor {
    
    /**
     * 获取BeanFactory中所有的EventListenerFactory类型的Bean，并按照@Order或Ordered接口中的      * 顺序进行排序。
     */
    @Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		this.beanFactory = beanFactory;

		Map<String, EventListenerFactory> beans = beanFactory.getBeansOfType(EventListenerFactory.class, false, false);
		List<EventListenerFactory> factories = new ArrayList<>(beans.values());
		AnnotationAwareOrderComparator.sort(factories);
		this.eventListenerFactories = factories;
	}
    
    /**
     * 当所有的单例bean都初始化完成后触发
     */
    @Override
	public void afterSingletonsInstantiated() {
        ConfigurableListableBeanFactory beanFactory = this.beanFactory;
		Assert.state(this.beanFactory != null, "No ConfigurableListableBeanFactory set");
        // 获取BeanFactory中的所有bean名称，从所有的bean中去解析需要注册到ApplicationContext中的事件监听器
		String[] beanNames = beanFactory.getBeanNamesForType(Object.class);
        for (String beanName : beanNames) {
            // ...省略一些判断是否排除的逻辑代码
            try {
                // 对通过检查的所有bean进行处理
                processBean(beanName, type);
            }
            catch (Throwable ex) {
                throw new BeanInitializationException("Failed to process @EventListener annotation on bean with name '" + beanName + "'", ex);
            }
        }
    }
    
    private void processBean(final String beanName, final Class<?> targetType) {
        // 第一行是进行去重的，同一个targetType只会处理一次
        // 第二行是排除一些特殊的bean，以"java"开头的bean
        // 第三行是排除那些没有使用@Component及其派生注解所注解的targetType
		if (!this.nonAnnotatedClasses.contains(targetType) &&
				!targetType.getName().startsWith("java") &&
				!isSpringContainerClass(targetType)) {

            // 用于存放从targetType中解析出来的所有被@EventListener及其派生注解所注解的方法
			Map<Method, EventListener> annotatedMethods = null;
            // ...省略不重要的代码
            for (Method method : annotatedMethods.keySet()) {
                for (EventListenerFactory factory : factories) {
                    // 查找支持method的EventListenerFactory来创建ApplicationListener
                    if (factory.supportsMethod(method)) {
                        Method methodToUse = AopUtils.selectInvocableMethod(method, context.getType(beanName));
                        ApplicationListener<?> applicationListener =
                            factory.createApplicationListener(beanName, targetType, methodToUse);
                        // 如果创建的ApplicationListener是ApplicationListenerMethodAdapter则对它进行初始化，传入ApplicationContext和EventExpressionEvaluator。
                        if (applicationListener instanceof ApplicationListenerMethodAdapter) {
                            ((ApplicationListenerMethodAdapter) applicationListener).init(context, this.evaluator);
                        }
                        // 将ApplicationListener加入ApplicationContext
                        context.addApplicationListener(applicationListener);
                        break;
                    }
                }
            }
        }
    }
    
}
```

==从上述代码可以得到编写 `@EventListener` 的一些注意项==

- 所在的类必须使用 `@Component` 或其派生注解来注解。
- 可以存在不同的 `EventListenerFactory`，可以针对不同的 `Method` 使用不同的 `EventListenerFactory`，而且会按照 `EventListenerFactory` 的顺序查找第一个来进行创建。

# 默认`ApplicationListener`工厂

完整类名：`org.springframework.context.event.DefaultEventListenerFactory`。

```java
public class DefaultEventListenerFactory implements EventListenerFactory, Ordered {

	private int order = LOWEST_PRECEDENCE;


	public void setOrder(int order) {
		this.order = order;
	}

	@Override
	public int getOrder() {
		return this.order;
	}


	public boolean supportsMethod(Method method) {
		return true;
	}

	@Override
	public ApplicationListener<?> createApplicationListener(String beanName, Class<?> type, Method method) {
        // 创建的ApplicationListener实例
		return new ApplicationListenerMethodAdapter(beanName, type, method);
	}

}
```

# `@EventListener` 注解

完整类名：`org.springframework.context.event.EventListener`。

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface EventListener {
    @AliasFor("classes")
	Class<?>[] value() default {};
    
    // 可以指定监听的事件类型
    @AliasFor("value")
	Class<?>[] classes() default {};
    
    // 可以指定SpEL监听条件
    String condition() default "";
}
```

# `ApplicationListenerMethodAdapter`

完整类名：`org.springframework.context.event.ApplicationListenerMethodAdapter`。

```java
public class ApplicationListenerMethodAdapter implements GenericApplicationListener {
    private final String beanName;
    private final Method method;
    // 监听的事件类型，从@EventListener#value()/classes()中解析而来
    private final List<ResolvableType> declaredEventTypes;
	// SpEL监听条件
	@Nullable
	private final String condition;
	// 监听顺序
	private final int order;
    
    @Override
	public void onApplicationEvent(ApplicationEvent event) {
		processEvent(event);
	}
    
    public void processEvent(ApplicationEvent event) {
        // 从event中解析出@EventListener注解的方法的参数值列表
		Object[] args = resolveArguments(event);
        // 根据condition判断是否需要处理
		if (shouldHandle(event, args)) {
            // 使用解析出来的参数值列表调用beanName指定的bean对象的method方法，可以拥有返回值。
			Object result = doInvoke(args);
			if (result != null) {
                // 如果返回值不为null则处理该返回值，会将返回值作为新的事件进行发布。
				handleResult(result);
			}
			else {
				logger.trace("No result object given - no result to handle");
			}
		}
	}
    
    protected void handleResult(Object result) {
        // 如果返回值是数组，会将数组中的每个元素作为一个事件进行发布
		if (result.getClass().isArray()) {
			Object[] events = ObjectUtils.toObjectArray(result);
			for (Object event : events) {
				publishEvent(event);
			}
		}
        // 如果返回值是集合，会将集合中的每个元素作为一个事件进行发布
		else if (result instanceof Collection<?>) {
			Collection<?> events = (Collection<?>) result;
			for (Object event : events) {
				publishEvent(event);
			}
		}
        // 否则就将返回值作为一个事件进行发布。
		else {
			publishEvent(result);
		}
	}
    
}
```

==从上面代码可以看出一些需要注意的地方==

- `@EventListener` 注解的方法可以有返回值，而且返回会作为新的事件进行发布。