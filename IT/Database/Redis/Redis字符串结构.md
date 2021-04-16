### String 类型具体是怎么保存数据的呢？
- 当保存 `64` 位有符号整数时，`String` 类型会把它保存为一个 `8` 字节的 `Long` 类型整数，这种保存方式通常也叫作 `int` 编码方式。
- 当保存的数据中包含字符时，`String` 类型就会用简单动态字符串（`Simple Dynamic String，SDS`）结构体来保存，如下图：

![Redis的SDS结构](../../Picture/Redis的SDS结构.jpeg)

- `buf`：字节数组，保存实际数据。为了表示字节数组的结束，`Redis` 会自动在数组最后加一个“`\0`”，这就会额外占用 `1` 个字节的开销。
- `len`：占 `4` 个字节，表示 `buf` 的已用长度。
- `alloc`：也占个 `4` 字节，表示 `buf` 的实际分配长度，一般大于 `len`。

在 `SDS` 中，`buf` 保存实际数据，而 `len` 和 `alloc` 本身其实是 SDS 结构体的额外开销。对于 `String` 类型来说，除了 `SDS` 的额外开销，还有一个来自于 `RedisObject` 结构体的开销。

### RedisObject 结构

因为 `Redis` 的数据类型有很多，而且，不同数据类型都有些相同的元数据要记录（比如最后一次访问的时间、被引用的次数等），所以，`Redis` 会用一个 `RedisObject` 结构体来统一记录这些元数据，同时指向实际数据。

一个 `RedisObject` 包含了 `8` 字节的元数据和一个 `8` 字节指针，这个指针再进一步指向具体数据类型的实际数据所在。

![RedisObject具体结构](../../Picture/RedisObject具体结构.jpeg)

为了节省内存空间，`Redis` 还对 `Long` 类型整数和 `SDS` 的内存布局做了专门的设计。

- 一方面，当保存的是 `Long` 类型整数时，`RedisObject` 中的指针就直接赋值为整数数据了，这样就不用额外的指针再指向整数了，节省了指针的空间开销。
- 另一方面，当保存的是字符串数据，并且字符串小于等于 `44` 字节时，`RedisObject` 中的元数据、指针和 `SDS` 是一块连续的内存区域，这样就可以避免内存碎片。这种布局方式也被称为 `embstr` 编码方式。
- 当字符串大于 `44` 字节时，`SDS` 的数据量就开始变多了，`Redis` 就不再把 `SDS` 和 `RedisObject` 布局在一起了，而是会给 `SDS` 分配独立的空间，并用指针指向 `SDS` 结构。这种布局方式被称为 `raw` 编码模式。

### Redis 的三种编码模式：int、embstr、raw 
![RedisObject三种编码模式示意图](../../Picture/RedisObject三种编码模式示意图.jpeg)

### Redis 的 dictEntry 结构体
`Redis` 会使用一个全局哈希表保存所有键值对，哈希表的每一项是一个 `dictEntry` 的结构体，用来指向一个键值对。`dictEntry` 结构中有三个 8 字节的指针，分别指向 `key`、`value` 以及下一个 `dictEntry`，三个指针共 2`4 `字节，如下图所示：

![Redis的dictEntry结构体](../../Picture/Redis的dictEntry结构体.jpeg)

三个指针只有 `24` 字节，为什么会占用了 `32` 字节呢？这就要提到 `Redis` 使用的内存分配库 `jemalloc` 了。

### Redis 使用的内存分配库 jemalloc
`jemalloc` 在分配内存时，会根据我们申请的字节数 `N`，找一个比 `N` 大，但是最接近 `N` 的 `2` 的幂次数作为分配的空间，这样可以减少频繁分配的次数。