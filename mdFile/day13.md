# 创建新进程

- 与fork不同,exec函数不是创建调用进程的子进程,而是创建一个新的进程
  取代调用进程自身。新进程会用自己的全部地址空间。覆盖调用进程的地址空间,但进程的PID保持不变

  ![fork与exec](./images/fork与exec.png)

- exec不是一个函数而是一堆函数(共6个),一般称为exec函数族。它们的功
  能是一样的,用法也很相近,只是参数的形式和数量略有不同

​			

```c
#include <unistd.h>
// 创建新进程来取代旧进程
int execl (const char* path, const char* arg, ... );
// 第一个参数传递可执行程序的路径
// 如 “/home/tarean/a.out”
// 第二个参数 传递a.out 需要的参数 ，第一个参数为可执行程序的名字
// 输入完所有参数记得，在末尾加上NULL
// execl("/home/tarena/a.out","a.out","hello.c","123",NULL);

int execlp (const char* file, const char* arg, ... );
// execlp 中的 p代表PATH环境变量
// 即给个文件名，execlp会在PATH中寻找，找不到则报错
int execle (const char* path, const char* arg, ... ,char* const envp[]);
// envp可以为变身之后的可执行程序，指定环境变量内容

int execv (const char* path, char* const argv[]);
// v版本与l的区别 v传数组 l一个一个传
int execvp (const char* file, char* const argv[]);
int execve (const char* path, char* const argv[],char* const envp[]);
```

-  其实6个exec函数中只有execve是真正的系统调用,其它5个函数不过是对
  execve函数的简单包装

  ![6个exec函数的本质](./images/6个exec函数的本质.png)

- 调用exec函数不仅改变调用进程的地址空间和进程映像,调用进程的一些属
  性也发生了变化

  >任何处于阻塞状态的信号都会丢失
  >被设置为捕获的信号会还原为默认操作
  >有关线程属性的设置会还原为缺省值
  >有关进程的统计信息会复位
  >与进程内存相关的任何数据都会丢失,包括内存映射文件
  >标准库在用户空间维护的一切数据结构(如通过atexit或on_exit函数注册的退出处理函数)
  >都会丢失

- 但也有些属性会被新进程继承下来,比如PID、PPID、实际用户ID和实际组ID、优先级,以及文件描述符等
- 注意如果进程创建成功,rexec函数是不会返回的,因为成功的exec调用会以
  跳转到新进程的入口地址作为结束,而刚刚运行的代码是不会存在于新进程的地址空间中的。但如果进程创建失败,exec函数会返回-1

## fork + exec模式

- 调用exec函数固然可以创建出新的进程,但是新进程会取代原来的进程。如
  果既想创建新的进程,同时又希望原来的进程继续存在,则可以考虑
  fork+exec模式,即在fork产生的子进程里调用exec函数,新进程取代了子进
  程,但父进程依然存在

​	![fork+exec](./images/fork+exec.png)

# system

```c
#include <stdlib.h>

int system (const char* command);
// 功能:执行shell命令
// 参数:command shell命令行字符串
// 返回值:成功返回command进程的终止状态,失败返回-1

// system函数执行command参数所表示的命令行,并返回命令进程的终止状态
// 若command参数取NULL,返回非0表示Shell可用,返回0表示Shell不可用
```

- 在system函数内部调用了vfork、exec和waitpid等函数

  >如果调用vfork或waitpid函数出错,则返回-1
  >如果调用exec函数出错,则在子进程中执行exit(127)
  >如果都成功,则返回command进程的终止状态(由waitpid的status参数获得)
  
- 使用system函数而不用vfork+exec的好处是,system函数针对各种错误和
  信号都做了必要的处理,而且system是标准库函数,可跨平台使用

## fork vs vfork

父进程通过vfork创建子进程后，没有数据复制的过程，子进程与父进程共用一份数据，子进程可以修改父进程数据

由于vfork不用再复制一份父进程的数据，因此一般使用vfork+exec的组合，省去了复制。exec以后，子进程便不会再修改父进程数据
