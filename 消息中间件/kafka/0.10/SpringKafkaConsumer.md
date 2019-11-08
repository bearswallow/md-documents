# 注解

## @KafkaListener

完整类名：`org.springframework.kafka.annotation.KafkaListener`。

```java
@Target({ ElementType.TYPE, ElementType.METHOD, ElementType.ANNOTATION_TYPE })
@Retention(RetentionPolicy.RUNTIME)
@MessageMapping
@Documented
@Repeatable(KafkaListeners.class)
public @interface KafkaListener {
    // container的唯一标识
    String id() default "";
    // container工厂类名称，如果未指定则使用默认的container工厂
    String containerFactory() default "";
    // 监听的topic数组。可以是字符串常量、property占位符和SpEL表达式。
    // 但是最终都要解析成topic
    String[] topics() default {};
    // 监听的topic模式。可以是字符串常量、property占位符和SpEL表达式。
    String topicPattern() default "";
    // 分配给该监听器的partition数组，与topics和topicPattern互斥
    TopicPartition[] topicPartitions() default {};
    // 消费者群组名称。用于将该监听器加入该名称的监听器
    String group() default "";
}
```

设置 `topics` 和 `topicPattern` 就等于开启了 KafkaConsumer 的 subscription 模式。

设置 `topicPartitions` 就等于开启了 KafkaConsumer 的 assign 模式。

可以在类或者方法上注解

- 在类上注解，则该类下的有 `KafkaHandler` 注解的方法都会处理 Kafka 消息。且这些方法都会使用相同的 `KafkaListenerContainerFactory` 创建 KafkaConsumer。
  - 当一条消息到达时，会依据各个方法的payload类型来选择相应的方法进行处理。
  - 可接受的参数类型
    - `org.apache.kafka.clients.consumer.ConsumerRecord`：原始的Kafka消息。
    - `org.springframework.kafka.support.Acknowledgment`：可进行手动ACK。
    - `@Payload`：添加到方法参数上的注解，Kafka消息正文内容，包含校验。
    - `@Header`：添加到方法参数上的注解，Kafka消息头部中的指定属性。
    - `@Headers`：添加到方法参数上的注解，Kafka消息的所有头部属性。
- 在方法上注解，则该方法会使用设定的 `KafkaListenerContainerFactory` 来创建 KafkaConsumer。
- 一个类或方法上可以添加多个 `@KafkaListener` 注解 `@KafkaListeners`。

### @KafkaLisenters

完整类名：`org.springframework.kafka.annotation.KafkaListeners`。

```java
@Target({ ElementType.TYPE, ElementType.METHOD, ElementType.ANNOTATION_TYPE })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface KafkaListeners {
	KafkaListener[] value();
}
```

## @KafkaHandler

完整类名：`org.springframework.kafka.annotation.KafkaHandler`。

```java
@Target({ ElementType.METHOD, ElementType.ANNOTATION_TYPE })
@Retention(RetentionPolicy.RUNTIME)
@MessageMapping
@Documented
public @interface KafkaHandler {
}
```

这只是一个标记注解，配置类上 `@KafkaListener` 一起使用。

## KafkaListenerAnnotationBeanPostProcessor

完整类名：`org.springframework.kafka.annotation.KafkaListenerAnnotationBeanPostProcessor`。

基于 Spring 的 BeanPostProcessor 来对 `@KafkaListener` 和 `@KafkaHandler` 进行处理，将这些方法注册到 kafka 时间监听器 container 中，以便接收到 Kafka 消息时调用响应的方法进行处理。

```java
public class KafkaListenerAnnotationBeanPostProcessor<K, V>
		implements BeanPostProcessor, Ordered, BeanFactoryAware, SmartInitializingSingleton {
    // 默认的KafkaListenerContainer工厂的FactoryBean名称
    public static final String DEFAULT_KAFKA_LISTENER_CONTAINER_FACTORY_BEAN_NAME = "kafkaListenerContainerFactory";
    // 记录所有类中的方法不含@KafkaListener注解的类，避免重复处理。
    private final Set<Class<?>> nonAnnotatedClasses =
			Collections.newSetFromMap(new ConcurrentHashMap<Class<?>, Boolean>(64));
    private KafkaListenerEndpointRegistry endpointRegistry;
    private final KafkaListenerEndpointRegistrar registrar = new KafkaListenerEndpointRegistrar();
    private final KafkaHandlerMethodFactoryAdapter messageHandlerMethodFactory =
			new KafkaHandlerMethodFactoryAdapter();
    // 用于解析 @KafkaListener 和 @KafkaHandler 中的SpEL表达式
    private BeanExpressionResolver resolver = new StandardBeanExpressionResolver();
    
    @Override
	public void afterSingletonsInstantiated() {
        // 将BeanFactory、KafkaListenerEndpointRegistry、containerFactoryBeanName 同步设置到KafkaListenerEndpointRegistrar中。
        // 从BeanFactory中获取所有的 KafkaListenerConfigurer，调用他们对KafkaListenerEndpointRegistrar进行配置。
        // 将KafkaListenerEndpointRegistrar中的MessageHandlerMethodFactory同步到KafkaHandlerMethodFactoryAdapter中。
        // 调用KafkaListenerEndpointRegistrar#afterSingletonsInstantiated
    }
    
    @Override
	public Object postProcessAfterInitialization(final Object bean, final String beanName) throws BeansException {
        // 不需要对已经被识别为不含@KafkaListener注解的类进行处理
        if (!this.nonAnnotatedClasses.contains(bean.getClass())) {
            // 获取类的 @KafkaListener 注解以及该类中含@KafkaHandler的方法
            // 获取类中含@KafkaListener注解的方法
            
            // 对类中含@KafkaListener注解的方法进行处理
            if (annotatedMethods.isEmpty()) {
				this.nonAnnotatedClasses.add(bean.getClass());
				if (this.logger.isTraceEnabled()) {
					this.logger.trace("No @KafkaListener annotations found on bean type: " + bean.getClass());
				}
			}
			else {
				// Non-empty set of methods
				for (Map.Entry<Method, Set<KafkaListener>> entry : annotatedMethods.entrySet()) {
					Method method = entry.getKey();
					for (KafkaListener listener : entry.getValue()) {
						processKafkaListener(listener, method, bean, beanName);
					}
				}
				if (this.logger.isDebugEnabled()) {
					this.logger.debug(annotatedMethods.size() + " @KafkaListener methods processed on bean '"
							+ beanName + "': " + annotatedMethods);
				}
			}
            
            // 对含@Kafka注解的类进行处理
			if (hasClassLevelListeners) {
				processMultiMethodListeners(classLevelListeners, multiMethods, bean, beanName);
			}
        }
        
        return bean;
    }
    
}
```

==有疑问的地方==

- 为什么 `nonAnnotatedClasses` 中只记录类中的方法不含 `@KafkaListener` 注解的类，而不用管类是否包含 `@KafkaListener` 注解。

# EndPoint

## KafkaListenerEndPoint

完整类名：`org.springframework.kafka.config.KafkaListenerEndpoint`。

```java
public interface KafkaListenerEndpoint {
    
    String getId();
    // 该EndPoint所在的分组，如果没有在任何一个分组里则返回null。
    String getGroup();
    Collection<String> getTopics();
    Collection<TopicPartitionInitialOffset> getTopicPartitions();
    Pattern getTopicPattern();
    // 
    void setupListenerContainer(MessageListenerContainer listenerContainer, MessageConverter messageConverter);
}
```

## AbstractKafkaListenerEndpoint

完整类名：`org.springframework.kafka.config.AbstractKafkaListenerEndpoint`。

```java
public abstract class AbstractKafkaListenerEndpoint<K, V>
		implements KafkaListenerEndpoint, BeanFactoryAware, InitializingBean {
    // 会对topics、topicPattern进行SpEL解析
    private BeanExpressionResolver resolver;
	private BeanExpressionContext expressionContext;
    // 消息过滤策略
    private RecordFilterStrategy<K, V> recordFilterStrategy;
    private boolean ackDiscarded;
    // 说明会在该层次进行重试处理
	private RetryTemplate retryTemplate;
    // 当所有重试都失败后的回调处理，可以作为fallback处理
	private RecoveryCallback<? extends Object> recoveryCallback;
    // 是否进行Kafka消息的批量处理
	private boolean batchListener;
    
    @Override
	public void afterPropertiesSet() {
        // 检查topics、topicPattern和topicPartitions之间的关系是否满足限制条件。
    }
    
    // 由子类来确定如何创建MessagingMessageListenerAdapter
    protected abstract MessagingMessageListenerAdapter<K, V> createMessageListener(MessageListenerContainer container,
			MessageConverter messageConverter);
    
    private void setupMessageListener(MessageListenerContainer container, MessageConverter messageConverter) {
        // 调用子类实现方法创建MessagingMessageListenerAdapter
        Object messageListener = createMessageListener(container, messageConverter);
        // 根据是否存在retryTemplate和recordFilterStrategy以及messageListener类型来创建不同的MessagingMessageListenerAdapter适配器。
        // 将MessagingMessageListenerAdapter注册进MessageListenerContainer
        container.setupMessageListener(messageListener);
    }
}
```

### RecordFilterStrategy

完整类名：`org.springframework.kafka.listener.adapter.RecordFilterStrategy`。

`ConsumerRecord` 过滤策略。在Kafka消息发送给某个@KafkaListener或@KafkaHandler注解的方法之前进行过滤。

```java
public interface RecordFilterStrategy<K, V> {
    boolean filter(ConsumerRecord<K, V> consumerRecord);
}
```

# 消息监听器

## KafkaDataListener

完整类名：`org.springframework.kafka.listener.KafkaDataListener`。

它是一个标记接口，container需要通过它来判断某个对象是否实现了该接口。

## GenericMessageListener

完整类名：`org.springframework.kafka.listener.GenericMessageListener`。

```java
public interface GenericMessageListener<T> extends KafkaDataListener<T> {
    void onMessage(T data);
}
```

container中使用的自动提交offset的消息监听器的顶层接口。

- `org.springframework.kafka.listener.MessageListener`：单条消息自动offset提交监听器接口。
- `org.springframework.kafka.listener.BatchMessageListener`：批量消息自动offset提交监听器接口。

## GenericAcknowledgingMessageListener

完整类名：`org.springframework.kafka.listener.GenericAcknowledgingMessageListener`。

```java
public interface GenericAcknowledgingMessageListener<T> extends KafkaDataListener<T> {
    void onMessage(T data, Acknowledgment acknowledgment);
}
```

container中使用的手动提交offset的消息监听器的顶层接口。

- `AcknowledgingMessageListener`：单条消息手动offset提交监听器接口。
- `BatchAcknowledgingMessageListener`：批量消息手动offset提交监听器接口。

# 消息监听器container

# 异常处理