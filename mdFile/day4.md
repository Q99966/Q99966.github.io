# 虚拟地址空间

比如32位操作系统虚拟地址空间范围是4G

### 那么虚拟地址空间范围有什么用？

​	每个进程都会有一张存放虚拟地址与实际物理地址的一张映射表，那么虚拟地址空间的范围也就是则张映射表的范围，现在在32位系统上可能担心这张表不够用。

但在64位上完全不用担心，64位操作系统的虚拟地址空间范围就大的没边了。

### 先有物理地址再有虚拟的地址的分配

​	当进程需要占用物理地址时，内核会为该进程载入内存，并提供一些虚拟地址。

### 那么就无法调用内核了吗？

​	用户地址空间的代码不能直接访问内核空间的代码和数据，但可以通过系统调用进入内核态，间接与系统内核交互。

## 作用

- 安全，无法直接获取物理地址。当进程调用虚拟地址时，内核会从那张映射表中查询不到就会报错，因此一个进程只能调用当前进程内的虚拟地址。也就达成了一个进程只能调用该进程所占用的物理地址。

  

- 方便，不需要计算物理地址

## 虚拟地址空间范围具体划分

### 内核与用户的划分

以32位举例

<img src=".\images\虚拟地址内核空间范围与用户空间范围.png" alt="虚拟地址内核空间范围与用户空间范围" style="zoom:150%;" />

如这张图上半部分是内核虚拟地址范围，下半部分是用户虚拟地址空间范围。



### 用户的具体划分

<img src=".\images\虚拟地址用户的具体划分.png" alt="虚拟地址用户的具体划分" style="zoom: 67%;" />

# 内存壁垒

不同进程，相同虚拟地址物理地址不同。

- 每个进程的用户空间都是0~3G-1,但它们所对应的物理内存却是各自独立
  的,系统为每个进程的用户空间维护一张专属于该进程的内存映射表,记录
  虚拟内存到物理内存的对应关系,因此在不同进程之间交换虚拟内存地址是毫无意义的

- 所有进程的内核空间都是3G~4G-1,它们所对应的物理内存只有一份,系统
  为所有进程的内核空间维护一张内存映射表init_mm.pgd,记录虚拟内存到
  物理内存的对应关系,因此不同进程通过系统调用所访问的内核代码和数据是同一份

- 用户空间的内存映射表会随着进程的切换而切换，内核空间的内存映射表则无需随着进程的切换而切换

  ![内存壁垒](./images/内存壁垒.png)

# 段错误

一切对虚拟内存的越权访问，都会导致段错误

- 试图访问没有映射到物理内存的虚拟内存
- 试图以非法方式访问虚拟内存，如对只读内存做写操作等。

​	这里写操作不是指下面这种

```c
const int i = 1;
i = 2;
```

​	运行这样的在编译阶段就会报错，不会发生段错误

​	反而通过下面代码可以修改i的值，并且不会报错

```c
const int i = 1;
*(int *)&i = 2;
```

​	这是因为const int 是非静态局部变量，位于栈区，是可以对内存做修改的。以及这里的只读内存是指代码区虚拟内存。

​	

```c
const static int i = 1;
*(int *)&i = 2;
```

​	运行上面代码则会报段错误，因为const static int 位于代码区
