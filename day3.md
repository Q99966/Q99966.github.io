



# 动态库的动态加载

## 背景

- 对于长时间运行的一个进程，在这段运行时间中这个库可能就调用了一两次，这样就造成了内存浪费。如何解决这个问题，这就需要我们在编译阶段前做点措施。于是就产生了几个操作动态库载入与释放的函数。

## 目的

- 在程序执行的过程中，开发人员可以动态加载共享库（什么时候用什么时候加载）

## 需求

- 在程序中动态加载共享库需要调用一组特殊的函数，它们被声明于一个专门的头文件中，并在一个独立的库中予以实现。
- 使用这组函数需要包含此头文件，并链接该库
  - #include<dlfcn.h>
  - -ldl

例如编译main.c

```shell
gcc main.c -ldl -o main
```



## <dlfcn.h>中的几个重要的函数

### dlopen

![dlopen](.\images\dlopen.png)

### dlsym

![dlsym](.\images\dlsym.png)

例如

```c
// 以下是一个函数指针，void* 能转换为任意指针
int (*add)(int,int) = dlsym(handle,"add");
```



### dlclose

![dlclose](.\images\dlclose.png)

### [dlerror](https://www.bing.com) 
![dlerror](.\images\dlerror.png)

## 辅助工具

### 查看符号表：nm

​	列出目标文件，可执行程序，静态库，或动态库中的符号

例如：nm libmath.a

### 查看依赖：ldd

​		查看可执行文件或者动态库所依赖的动态库

例如：ldd a.out

# 错误处理

## 需要提前了解的几个概念

### 输出缓冲区

从缓冲区到显示屏上满足三个条件里面其中一个就行

- 有\n
- 缓冲区满了
- 程序正常结束

### 三个特殊的文件

| 文件名 |   类型   |    默认形式    |
| :----: | :------: | :------------: |
| stdin  | 标准输入 |  默认键盘输入  |
| stdout | 标准输出 | 默认显示器输出 |
| stderr | 标准错误 | 默认显示器输出 |

标准输出有缓冲区，而标准错误没有

## fprintf 

### 作用

- 向一个文件中输入一串文字

### 格式

- fprintf(fp,"需要输入的串");
- 例如 fprintf(stderr,"dlopen:%s\n",dlerror());



### 为什么有时候用fprintf而不用printf?原因如下。

- printf 需要经过缓冲区才能输出到屏幕上，因此有时候并不能立马把错误显示在屏幕上
- 但是fprintf它能出现错误，立马显示在屏幕上，只需要让它写入stderr文件中，即fprintf(stderr,"dlopen:%s\n",dlerror());

## 第一种方法

### 通过错误号了解具体的错误原因

- 系统定义的整数类型全局变量errno中储存了最近一次系统调用的错误编号

- 头文件errno.h中包含了对errno全局变量的外部声明和各种错误号的宏定义
- /usr/include/errno.h
- /usr/include/asm-generic/errno.h   // 包含35-132错误定义
- /usr/include/asm-generic/errno-base.h   // 包含1-34错误定义

### 注意错误号非零不能当作错出现的标志

因为当发生第一次错误后errno就不在等于零了

```c
//错误处理
#include<stdio.h>
#include<string.h>// strerror();
#include<stdlib.h>
#include<errno.h> // 全局变量 errno

int main(void){
    void* p = malloc(0xffffffffffffffff);
    /*if(p == NULL){//malloc 失败-->原因-->编号-->errno
        fprintf(stderr,"errno = %d\n",errno);
        fprintf(stderr,"malloc:%s\n",strerror(errno));
        perror("malloc");
        return -1;
    }
    free(p);*/
    FILE*fp = fopen("test.c","r");
    //if(errno){
    if(fp == NULL){
        perror("fopen");
        return -1;
    }
    fclose(fp);
    return 0;
}

```

比如该上述代码中，malloc发生的错误，在fopen处也会提示出错，这样就会出现错误误判。

### 解决记不住错误编号对应错误名的问题

​	在<string.h>头文件当中有一个可以将整数形式的错误号转换为有意义的字符串

```c
#include<string.h>
char* strerror(int errnum)
```

- 参数：errnum 错误号
- 返回值：返回与参数错误号对应的描述字符串

## 第二种方法

​	使用perror输出错误

```c
#include<stdio.h>
void perror(char const* tag);
```

- 功能：在标准出错设备上打印最近一次函数调用的错误信息
- 参数：tag 为用户自己制定的提示内容，输出时，会自动在该提示内容和错误信息之间添加冒号进行分隔

## 为什么要用以上的方法来显示错误？

​	因为只有少部分函数出现错误才会才会导致程序的中断，并返回错误。而大部分函数只会返回一个参数，并不会中断程序，因此需要自己手动来添加错误提示。

# 玩指针十二字真言

**由近及远**

**从右向左** 

**括号优先**        

```c
const int N = 100;
int *p[N]; // 一个数组存放指针
int (*p)[N]; // 一个指针，指向一个数组
int (*add)(int,int) // 一个函数指针
```



