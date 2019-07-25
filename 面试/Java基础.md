# 数据结构

## `Map`

### 非线程安全

参考文章：

- [HashMap中capacity、loadFactor、threshold、size等概念的解释](https://blog.csdn.net/fan2012huan/article/details/51087722)

- [深入浅出 Map 的实现（HashMap、HashTable、LinkedHashMap、TreeMap）](https://blog.csdn.net/moneyshi/article/details/50593243)

- [HashMap，LinkedHashMap与TreeMap](https://blog.csdn.net/weixin_43560292/article/details/86099207)

比较

- **`HashMap`**
  - 无序、key不可重复。
  - 允许存在一个key为 `null` 的项。
  - hash + 链表（哈希冲突）；hash + 红黑树（链表长度超过指定阈值时）。
  - 存在 `resize` 操作，当保存的数据项超过填充因子所规定的项数后需要进行 `resize` 操作，容量扩大为原来的 2 倍，对所有项进行重新哈希。
- **`LinkedHashMap`**
  - `key` 不可重复，根据 `accessOrder` 的值进行排序
    - `accessOrder` 为 `TRUE` 时，数据项按访问顺序存储；`put` 和 `get` 操作都会将相关的数据项移动到链表的末尾。==适合实现 `LRU` 算法，这时需要继承类覆盖 `removeEldestEntry` 方法。==
    - `accessOrder` 为 `FALSE` 时，数据项按更新顺序存储；`put` 操作会将相关的数据项移到链表的末尾。==适合实现不可重复的队列。==
  - 不存在 `resize` 操作。
- **`TreeMap`**
  - 基于 `key` 的顺序存储在红黑树中；key` 不可重复。
  - ==很适合执行一些顺序相关的操作：最大值、最小值、区间查询等。==
  - 不存在 `resize` 操作。

### 线程安全

参考文章

- [Java8 中 ConcurrentHashMap工作原理的要点分析](https://www.cnblogs.com/nullzx/p/8647220.html)
- [java-并发-ConcurrentHashMap高并发机制-jdk1.8](https://blog.csdn.net/jianghuxiaojin/article/details/52006118)
- [从2-3-4树到红黑树（上）](https://www.cnblogs.com/nullzx/p/6111175.html)

比较

- **`HashTable`**
  - 无序、key不可重复。
  - key和value都不允许为 `null`。
  - hash + 链表（哈希冲突）
  - 存在 `resize` 操作。
  - 线程安全，同时只能允许一个线程访问。
    - 通过将所有方法都定义为 `synchronized`， 效率太低，现在都会使用 `ConcurrentHashMap`。
- **`ConcurrentHashMap`**
  - 无序、key不可重复。
  - key和value都不允许为 `null`。
  - hash + 链表（哈希冲突）；hash + 红黑树（链表长度超过指定阈值时）。
  - 存在 `resize` 操作，且可多线程并发执行。
    - 因为 `hash` 桶的大小为 2 整数次幂，每次扩容为原来的两倍（还是 2 的整数次幂），原来 `hash` 桶中的元素经过重新 `hash` 后要么还保留在原来的位置（k），要么会转移到 原`hash`桶容量+k 的位置，所以==每个 `hash` 桶的 `resize` 操作都是相互独立的==，支持多线程并发执行。
    - 进行 `put`、`remove` 操作的线程如果发现要操作的 `hash` 桶正在执行 `resize` 操作，它就会先帮助完成其它桶的 `resize` 操作（根据并发执行线程数设置决定），然后再执行自身的操作。
    - 对某个 `hash` 桶执行 `resize` 操作完成时，会将该 `hash` 桶中的元素替换为 `ForwardingNode`，这样在整个 `Map` 完成 `resize` 操作之前也不会影响 `get` 操作获取元素。因为 `ForwardingNode` 会将查找操作转发给扩容后的 `Map`。
  - 线程安全
    - 主要通过 `CAS` 实现大部分线程安全。
    - 针对 `put`、`remove` 和 `resize` 操作通过对 `hash` 对应的桶加锁来实现这些操作的线程安全，所以执行上述操作的加锁粒度是单个 `hash` 桶，大大提高了多线程并发操作的性能。
    - 每次 `put`、`remove` 操作都会影响元素数量，`ConcurrentHashMap` 将元素数量的维护分割成一个数组来维护，每个线程会随机在数组中的某个元素上修改数量，每次需要获取元素数量时，都需要遍历该数组，将每个数组里的数量累加。该数组的规模会随着桶数量的规模动态增长。

## `Set`

`Set` 是一组不能重复的数据项集合。目前主要的实现都是基于 `Map` 的，数据项作为 `Map` 的 `key` 存储。

- `HashSet`：基于 `HashMap` 实现。
- `LinkedHashSet`：基于 `LinkedHashMap` 实现，没有 `accessOrder` 选项，所以无法实现 `LRU` 算法。
- `TreeSet`：基于 `TreeMap` 实现。

## `List`

