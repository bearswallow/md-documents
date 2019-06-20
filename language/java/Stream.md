# 概述

从Java SE 8 开始引入了Stream接口，它提供了一种抽象的编程模式，让开发人员主要集中于“做什么”，而不是“怎么做”。==要了解 `Stream` 要首先了解 `Spliterators`==。

## 与集合的区别

虽然它看起来很像集合，但是它与集合有几点显著的区别

- Stream 不会存储数据，数据还是存储在底层的集合或动态生成。
- Stream 不会修改源数据，所有转换操作都会产生新的 Stream。
- Stream 是延迟执行，只有在需要它的执行结果时才会执行。而且在得到结果后就立即终止，不会遍历Stream中的所有元素。

## 特例：Optional

`Optional` 可以看成具有0个或1个元素的 `Stream`，所以可以对它执行一些 `Stream` 上执行的操作。如：map、flatmap 等。

# 流程

Stream 提供了管道操作的模式，它主要包含下面三个阶段。

- 创建
- 转换：可以定义任意多个转换步骤，每个转换步骤都会生成新的 Stream。
- 终止操作：结束整个Stream的运行，有的终止操作会产生一个结果。

##  创建

```java
// 通过常量来创建
public static<T> Stream<T> of(T... values);
// 通过起始值和一元运算器来动态生成 Stream 中的元素
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f);
// 通过一个数据提供者来生成 Stream 中的元素
public static<T> Stream<T> generate(Supplier<T> s);
// 通过Builder来生成
public static<T> Builder<T> builder();
// 生成空Stream
public static<T> Stream<T> empty();
// 将两个Stream合并成一个Stream
public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b);
// ？？？
public static<T> Stream<T> of(T t);
```

其它能生成 Stream 的类

- `StreamSupport` 和 `Arrays` 都提供了生成 int、long 和 double 的Stream。
- `Arrays` 提供了将指定范围的数组元素作为 Stream 的方法。
- `Collection` 接口提供了 `stream` 和 `parallelStream`方法来生成 Stream。

## 转换

## 终止操作

终止操作主要分为几类

- 不需要输出结果的操作
- 判断存在性
- 查找单个元素
- 统计
- 求值
- 分组
- 将元素存储到其它数据结构中

### `Stream`

下面列出 `Stream` 的方法说明

- 不需要输出结果的操作：forEach、forEachOrdered。
  - 在非parallel模式下，forEach 和 forEachOrdered 访问元素的顺序是一样的；在 parallel 模式下，forEach 无法保证元素访问的顺序，如果这个时候采用 forEach 则会散失并发能力。
- 判断存在性：anyMatch、allMatch 和 nonMatch。
- 查找单个元素：findFirst、findAny、min、max、collect。
  - 在非parallel模式下，findFirst 和 findAny 得到的结果是一样的。
- 统计：count、min、max、reduce、collect。
- 求值：reduce、collect。
  - 这里的操作必须满足结合律，这样在 parallel 模式下才能正确执行。
  - reduce 的 `identity` 必须是线程安全的，否则在 parallel 模式下不能正确执行。这个时候需要使用 `collet` 。
- 分组：collect。
- 将元素存入其它数据结构：toArray、collect。

### `Collectors`

上述的大部分操作都可以通过 `Stream#collect` 方法实现，而在该方法中 `Collectors` 扮演了及其重要的角色。下面列出 `Collectors` 的方法说明。

- 查找单个元素：minBy、maxBy。
- 统计：counting、averaging[Int|Long|Double]、suming[Int|Long|Double]、summarizing[Int|Long|Double]。
- 求值：reducing。与 `Stream#reduce` 相对应。
- 分组：groupingBy、groupingByConcurrent、partitioningBy。
- 将元素存入其它数据结构：joining、toList、toSet、toMap、toConcurrentMap。

# 原始类型 Stream

对于 java 的原始类型 byte、short、int、long、float、double、boolen、char，如果使用普通的 Stream 则要面临的大量的装箱操作。为了提高性能，java se 提供了 IntStream、LongStream 和 DoubleStream 来处理原始类型Stream，这样就可以避免装箱操作。

原始类型与原始类型Stream的对应关系

- IntStream：byte、short、int、boolean、char。
- LongStream：long。
- DoubleStream：float、double。

原始类型Stream与普通Stream之间的相互转换

```java
// 普通Stream转IntStream
Stream<String> words = ...;
IntStream lenghts = words.mapToInt(String::length);
// IntStream转普通Stream
Stream<Integer> integers = IntStream.range(0, 100).boxed();
```

**原始类型Stream与普通Stream的不同之处**

- `toArray` 方法返回原始类型的数组。
- 返回 `Optional` 结果的方法会返回 `OptionalInt`、`OptionalLong`、`OpationDouble`。
- 提供额外的 sum、average、max 和 min 方法。
- 提供 summaryStatistics 方法。

# ParallelStream

注意事项

- 在进行转换之前需要将普通 `Stream` 转换为 `ParallelStream`。
- 需要处理大规模的数据且所有数据都在内存中。
- 数据能够很好分割成多个部分并发处理。
- 针对 `ParallelStream` 的操作都必须是线程安全的。
- `ParalleStream` 是通过 `fork-join` 线程池完成并发执行的，所以不能在执行操作中阻塞，要不然整个 `fork-join` 线程池中的线程将被消耗完，导致无法完成其它任务。