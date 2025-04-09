![操作系统分层](.\images\操作系统分层.png)

# 操作系统分层

## shell(壳)  

-  相当与系统语言(是用户与内核之间交互的翻译官)
- 在Linux中叫bash
- shell脚本不同于“01001”的机器语言，它直接由shell交给内核来执行

# 环境变量

## 进程（简单理解版）

- 正在执行的程序

## 环境变量表
![环境变量表](.\images\环境变量表.png)

- 本质是一个指针数组，存对应变量的地址

- 每个进程都有一张独立的环境变量表

- 用env 来查看环境变量

## 环境变量的类别

- 全局环境变量：当前shell和子进程都是可见的
- 局部环境变量：只有当前shell可见
- export name // 将局部环境变量设置成全局变量
- unset name //删除环境变量 

## PATH环境变量的作用

- 记录bash程序的检索路径
- 家目录下有.bashrc的脚本文件，每次bash启动都会执行该脚本
- 可以在脚本最低下写相应的命令，来修改之后所有的bash进程
- 执行source ~/.bashrc  来重启bash

# Linux内核源码

- [Linux source code (v6.12) - Bootlin Elixir Cross Referencer](https://elixir.bootlin.com/linux/v6.12/source)

# 在Linux离线安装gcc

[linux 安装gcc---GMP 4.2+, MPFR 3.1.0+ and MPC 0.8.0+ - burlingame - 博客园](https://www.cnblogs.com/zhiminyu/p/18267733)

# 遗留问题

- [ ] 高版本gcc没有安装成功
- [ ] 课件里面的二重指针没有搞懂



## 高版本gcc没有安装成功

​	看说明书好像跟一系列软件版本太低有关，包括gcc版本

​	目前gcc-4.8.5 支持c11,但是默认为c98,因此使用c11还得执行

- g++ -std=c++11 -o test test.cpp 

​	这样看起来这个问题目前没有这么重要

 

​	