#### 1.数组和切片有什么异同

切片是对数组的封装，两者都可以通过下标访问元素。

数组是定长的，定义后不能再更改，在Go语言中使用较少，不同长度的数组表示不同的类型。

切片可以动态扩容，类型和长度无关，它的数据结构包括三个部分：1.指向底层数组的指针`array` 2.长度`len` 3.容量`cap` ，共3*8=24字节，初始化时使用`make`创建，而数组只能使用`new`创建。


#### 1.切片的结构和扩容规则，操作后底层数组的变化

向切片中添加元素时，如果长度没有超过容量，则底层数组不会改变只是长度增加；如果超过容量发生扩容，会重新分配一块内存，然后把原数组的内容复制过来，再将新的元素添加。

扩容机制在1.18版本以前的规则是：切片长度小于1024时进行双倍扩容，大于1024时渐进式扩容，每次扩容当前容量的1/4。

1.18版本后为了让切片扩容更加平滑，优化了扩容规则 1.双倍扩容：切片当前容量小于256时，容量直接翻倍 2.渐进式扩容：切片当前长度大于256时，新容量公式为`newcap += (newcap + 3*threshold) /4`，等价于扩容`1/4+192`。

#### 2.map的结构和扩容原理

```Go
type hmap struct {
    count     int  //元素的个数
    flags     uint8  //状态标志
    B         uint8  // 可以最多容纳 6.5 * 2^B 个元素(kv选择buckets时用的是与运算的方法)，6.5为装载因子
    noverflow uint16 // 使用的溢出桶的数量
    hash0     uint32 // 随机的hash种子，每个map不一样，减少哈希碰撞的几率

    buckets    unsafe.Pointer // 指向一个bmap数组
    oldbuckets unsafe.Pointer // 如果存在扩容会有扩容前的buckets
    nevacuate  uintptr        // 迁移进度，小于nevacuate的buckets已迁移完毕

    extra *mapextra // 记录溢出桶相关信息
}
```

```Go
type mapextra struct {
   overflow    *[]*bmap  // 当前使用的溢出桶
   oldoverflow *[]*bmap  // 扩容阶段存储旧桶的溢出桶

   // nextOverflow holds a pointer to a free overflow bucket.
   nextOverflow *bmap  // 下一个空闲溢出桶
}
```

```Go
type bmap struct {
    tophash [bucketCnt]uint8
}
```

map的本质是哈希表，底层为一个`hmap`结构体，hmap中使用桶`bmap`(buckets)来承接数据，每个桶能存8组k/v。当有数据读写时，会用`key`的hash值找到对应的桶。为加速hash来定位桶，`bmap`里记录了`tophash`(hash的高8位)，`bmap`中为了数据排列更加紧凑，把8个`key`放在一起，8个`value`放在一起，`key`的前面是8个`tophash`，最下面是一个溢出桶的指针。

当一个key经过hash指向的桶已经存满了，就会在该`bmap`的后面链一个溢出桶(overflow)来解决此问题。

<img src="https://cdn.nlark.com/yuque/0/2025/png/22528385/1740239156808-6c716d86-c6eb-462e-aaae-41ead4f10f72.png?x-oss-process=image%2Fformat%2Cwebp" alt="image.png" style="zoom: 33%;" />

扩容方式

map的扩容方式为`渐进式扩容`，为了避免一次性扩容后带来的瞬时性能抖动，map扩容时会先分配足够多的新桶，用一个字段来记录要迁移的旧桶的位置，再用一个字段记录旧桶迁移进度，每次读写操作时检查是否存在未迁移的旧桶，若存在，则从旧桶中选取一部分数据迁移到新桶，同时更新迁移进度，直到所有数据迁移完毕。

扩容规则

存储键值对的数量 `count`

桶数量 `m = 2^B`

负载因子 `loadFactor = count/m `

翻倍扩容

负载因子的值超过6.5就会触发翻倍扩容，分配新桶的数目是旧桶的两倍

等量扩容

创建和旧桶数量一样多的新桶

loadFactor没超过6.5，但溢出桶数量过多时会触发等量扩容

- B不大于15时，溢出桶数量大于常规桶等量扩容
- B大于15时，溢出桶数量大于2^15等量扩容

原因是很多溢出桶中的键值对被删除，等量扩容经过重新排列后内存更加紧凑

#### 3.make和new的区别

在 Go 语言中，`make` 和 `new` 都是用于内存分配的关键字，但它们有着显著区别：

1. 适用类型

   `new`：仅用于分配内存，适用于`值类型`，`指针类型`和`接口类型`等，返回指向零值的指针

   `make`：专门用于创建并初始化`slice`、`map`和`channel`这三种引用类型。

2. 返回值

   `new`：返回指向已分配内存的指针

   `make`：直接返回初始化后的引用类型本身而非指针

3. 内存初始化

   `new`：分配的内存初始化为类型零值，不做额外初始化

   `make`：对切片初始化并按指定长度填充零值

#### 4.channel的结构和特性 



#### 5.各种情况下对channel进行读写会有什么结果



#### 6.值传递和引用传递的区别



#### 7.怎么实现一个任务只被执行一次



#### 8.进程/线程/协程（Goroutine）的区别



#### 9.CSP模型和常用并发机制



#### 10.GMP模型 work stealing/hand off



#### 11.Go语言在云原生领域的优点

交叉编译 二进制体积小 社区强大

#### 12.Goroutine的生命周期控制



#### 13.Context的用法和理解



#### 14.GC机制



#### 15.两个Goroutine交替打印

1个channel，两个goroutine创建后，向channel中写入数据，期望先打印的goroutine启动后读数据，后打印的启动后写数据，就实现了交替打印

2个channel，两个goroutine创建后，分别从自己对应的channel中读数据，并在结束时向对方的goroutine中写数据

#### 16.select机制



#### 17.内存逃逸分析



#### 18.接口耗时长如何排查



#### 19.Cond和Broadcas/Signal



#### 20.微服务架构的特点和优势



#### 21.防止Goroutine超时怎么处理



