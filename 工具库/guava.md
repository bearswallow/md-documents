# 注解

在 `com.google.code.findbugs:jsr305` 、 `com.google.errorprone:error_prone_annotation` 和 `com.google.guava:guava` 包中的很多注解都很适用，再无法使用 `java` 本身的机制对需要表达的意图进行规范化限制时，可以使用这些注解来增强代码的可读性，也会让维护者更好的保持这些意图。

比较重要的一些注解：

- 线程安全注解：`@Immutable`、`@ThreadSafe`、`@NotThreadSafe` 和 `@GuardedBy`。
- 对于需要关闭的资源的处理：`@WillClose`、`@WillCloseWhenClosed` 和 `@WillNotClose`。
- 专门为了测试而提升可访问性的注解：`@VisibleForTesting`。

# 基础功能

主要牵涉的包

- com.google.guava:guava -> com.google.common.base

`guava` 中提供的一些基础功能基本上目前都已经在 `Java8` 中得到了体现，所以比较建议采用 `Java8` 中提供的相应功能，它可以更好的使用 `Java8` 中引入的特性。

从下面的基础功能可以看出 `guava` 的先进性，因为它的很多思想最后都在会成为jdk的一部分。

## 不建议使用的功能

### Predicate

在 `guava` 中提供了 `Predicate` ，还提供了配套的 `Predicates`，它也提供了 `Java8` 中提供的聚合方式 `and`、`or` 和 `negate`，它还默认提供了一些常用的 `Predicate` 实现，如：`alwaysTrue`、`alwaysFalse`、`isNull`、`notNull`、`instanceOf`、`assignableFrom`、`in`、`containsPattern `等。

==在 `Java8` 中也提供了 `Predicate`，建议采用 `Java8` 中的 `Predicate`，它比 `guava` 提供的 `Predicate` 有更多的优点==：

- 自身提供了聚合方式 `and`、`or` 和 `negate`，可以更好的以流式的方式来组合多个 `Predicate`。
- 它是一个 `FunctionalInterface`，可以很好的使用 `lambda` 和 `方法引用` 这些 `Java8` 的新特性。
- 基于 `jdk` 本身的功能，更加通用。
- `guava` 中 `Predicates` 提供的大部分功能都可以使用 `方法引用` 来提供，比如：`isNull` -> `Objects::isNull`，使用起来更简洁。

### Function

在 `guava` 中提供了 `Function` 接口来表示接收一个参数并有返回值的函数。

在 `java8` 中也引入了相应的 `Function` 接口，而且就像 `Predicate` 一样，它是一个 `FunctionalInterface` 接口，也提供了 `compose`、`andThen` 等聚合方式，==拥有相同的优点，建议采用 `Java8` 中的 `Function`==。

### Supplier

请使用 `java8` 中的 `Supplier`。

### Optional及其子类

请使用 `java8` 中的 `Optional` 。

### Objects

请使用 `java8` 中的 `Objects`。

### MoreObjects

里面提供了一个 `ToStringHelper` 类，可以使用流式方式设置生成字符串的数据，==如果仅仅是为了生成指定对象的字符串，则可以使用 `lombok` 的 `@ToString` 来设置就好==，还省去了使用 `ToStringHelper` 的手动设置过程。

### Stopwatch

一个可以重复使用的计时器，目前大部分项目都会基于 `Spring` 开发，==建议使用 `Spring` 提供的 `StopWatch` 计时器，它支持分段计时。==

### Strings

一个字符串处理工具类，==建议使用 `Spring` 提供的 `StringUtils` 或者 `org.apache.commons.lang3` 提供的 `StringUtils` 。==

## 建议使用的功能

### Preconditions

里面提供了很多对传入的值进行检查并在不满足条件时抛出相应异常的工具方法，很适合用于参数检查等场景，可以简化代码。

### CharMatcher

提供了场景的字符串匹配器，可以与 `Spliter` 结合使用。

# 基础类型处理

牵涉的包

- com.google.guava:guava -> com.google.common.primitives

包含很多 `Xxxs` 的工具类

# 缓存

实现了一套完整的缓存方案。

## RemovalListener

可以监听缓存中的缓存被移除、替换、过期、回收或由于缓存容量填满时自动移除。通过 `RemovalCause` 来判断。

## LongAddable

这个是在 `jdk` 提供 `AtomicLong` 类型之前，guava实现的与其功能相似的类，目前直接使用 `AtomicLong` 即可。