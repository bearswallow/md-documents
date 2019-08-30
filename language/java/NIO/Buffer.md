`NIO` 中的 `Buffer` 是一个用于存储基本数据类型值的容器，它以类似于数组有序的方式来存储和组织数据。每个基本数据类型（除去 `boolean`）都有一个子类与之对应。

# 核心属性

`Buffer` 存在4个核心属性：

- capacity（容量）：容量在创建之后就不会改变了。它表示 `Buffer` 最多能存储多少数据。
- limit（限制）：表示 `Buffer` 中第一个不应该读取或写入元素的索引。
- position（位置）：表示 `Buffer` 中下一个要读取或写入元素的索引。
- mark（标记）：它是一个索引，在调用 `reset()` 方法时，会将 `Buffer` 的 position 位置重置为该索引。
- remaining（剩余空间）：直接通过 limit - position 来计算。

它们满足这样的关系：0 <= mark <= position <= limit <= capacity。

## 对核心属性的修改

- `clear()`：还原 `Buffer` 的初始状态（position=0, limit=capacity,mark=-1）。
  - 它并不会真正清除 `Buffer` 中数据，这个时候如果从 `Buffer` 中读取数据是能够得到之前的数据的。
  - 使用场景：在 `Buffer` 存储数据之前调用此方法。
- `flip()`：反转 `Buffer`（limit=position, position=0, mark=-1）。
  - 使用场景：在写入后进行读取之前；或者读取后需要进行写入之前调用此方法。
- `rewind()`：重绕 `Buffer`（position=0, mark=-1）。
  - 使用场景：为“重新读取”已包含的数据做好准备。
- `reset()`：将 `Buffer` 的 position 位置重置为 mark。如果 mark=-1 则会抛出异常。
- `limit(int)`：设置 limit，如果超过 capacity 或小于0则会抛出异常；如果newLimit < position，则设置 position = newLimit。 
- `position(int)`：设置 position，如果超过 limit 或小于0则会抛出异常。

重新设置后如果 mark > position 则设置 mark = -1。

# ByteBuffer

`ByteBuffer` 提供了6类操作。

- 以绝对位置和相对位置读写单个字节的 `get()` 和 `put()` 方法。
- 使用相对批量 `get(byte[] dst)` 方法可以将 `Buffer` 中的连续字节传输到 `byte[]` 目标数组中。
- 使用相对批量 `put(byte[] src)` 方法可以将 `byte[]` 数组或其他 `ByteBuffer` 中的连续字节存储到此缓冲区中。
- 使用绝对和相对 `getType`  和 `putType` 方法可以按照字节顺序在字节序列中读取其它基本类型的值。
  - 这些操作会进行数据类型的自动转换。
- 提供了创建视图缓冲区的方法，这些方法允许将字节缓冲区视为包含其他基本类型值的缓冲区。
  - asCharBuffer()、asDoubleBuffer()、asFloatBuffer、asIntBuffer、asLongBuffer、asShortBuffer。
- 提供了对字节缓冲区进行压缩（compacting）、复制（duplicating）和截取（slicing）的方法。
  - 创建一个新的 `ByteBuffer`，与原 `ByteBuffer` 共享底层数据，有独立的 capacity、limit、position 和 mark。

## 相对位置和绝对位置

- 所有的相对位置操作都会改变 `ByteBuffer` 的 position；绝对位置操作不会改变 `ByteBuffer` 的 position。
- 绝对位置操作的索引如果超出了 limit 的范围则会抛出异常。
- 写入操作时，如果需要写入的字节数大于字节缓冲区的remaining，则会抛出异常；读取操作时，如果需要读取的字节数大于字节缓冲区的remaining时，则会抛出异常。

## 创建方法

- 直接字节缓冲区：`ByteBuffer.allocDirect`。
  - 分配和释放所需的时间成本通常要高于堆内缓冲区，但是操作速度比堆内缓冲区快，并且可以使用零拷贝技术减少数据拷贝的开销。
  - 善于保存那些易受操作系统本机I/O操作影响的大量、长时间保存的数据。
- 堆内字节缓冲区：`ByteBuffer.alloc`、`ByteBuffer.wrap`。
  - `ByteBuffer.wrap` 是将一个 byte[] 包装成字节缓冲区，字节缓冲区直接使用该 `byte[]` 作为数据容器，所以修改 byte[] 或 字节缓冲区都会影响到另一个的内容。

