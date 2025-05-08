# 信号集

​	多个信号组成的信号集合谓之信号集

## 系统内核用sigset_t类型表示信号集

```c
// 在<signal.h>中又被定义为typedef
// 在<sigset.h>中有如下类型定义
#define SIGSET_NWORDS (1024 / (8 * sizeof (unsigned long int)))
typedef struct {
unsigned long int_val[_SIGSET_NWORDS];
}__sigset_t;
```

- sigset_t类型是一个结构体,但该结构体中只有一个成员,是一个包含32个
  元素的整数数组(针对32位系统而言)

- 可以把sigset_t类型看成一个由1024个二进制位组成的大整数

  >其中的每一位对应一个信号,其实目前远没有那么多信号
  >某位为1就表示信号集中有此信号,反之为0就是无此信号
  >当需要同时操作多个信号时,常以sigset_t作为函数的参数或返回值的类型

  ![sigset_t](./images/sigset_t.png)

## sigfillset

```c
#include <signal.h>
int sigfillset (sigset_t* sigset);
//  功能:填满信号集,即将信号集的全部信号位置1
//  参数:sigset 信号集
// 		返回值:成功返回0,失败返回-1
```

## sigemptyset

```c
#include <signal.h>

int sigemptyset (sigset_t* sigset);
// 功能:清空信号集,即将信号集的全部信号位清0
// 参数:sigset 信号集
// 		返回值:成功返回0,失败返回-1
```

## sigaddset

```c
#include <signal.h>

int sigaddset (sigset_t* sigset, int signum);
// 功能:加入信号,即将信号集中与指定信号编号对应的信号位置1
// 参数:sigset 信号集
// 		signum:信号编号
// 		返回值:成功返回0,失败返回-1
```

## sigdelset

```c
#include <signal.h>
int sigdelset (sigset_t* sigset, int signum);
// 功能:删除信号,即将信号集中与指定信号编号对应的信号位清0
// 参数:sigset 信号集
// 		signum:信号编号
// 		返回值:成功返回0,失败返回-1
```

## sigismember

```c
#include <signal.h>
int sigismember (const sigset_t* sigset, int signum);
// 功能:判断信号集中是否有某信号,即检查信号集中与指定信号编号对应的信号位是否为1
// 参数:sigset 信号集
// 		signum:信号编号
// 		返回值:有则返回1,没有返回0,失败返回-1
```



# 信号屏蔽

-  当信号产生时,系统内核会在其所维护的进程表中,为特定的进程设置一个
  与该信号相对应的标志位,这个过程就叫做递送(delivery)
- 信号从产生到完成递送之间存在一定的时间间隔,处于这段时间间隔中的信号状态称为未决(pending)
-  每个进程都有一个信号掩码(signal mask),它实际上是一个信号集,位于该
  信号集中的信号一旦产生,并不会被递送给相应的进程,而是会被阻塞(block)在未决状态

- 在信号处理函数执行期间,这个正在被处理的信号总是处于信号掩码中,如
  果又有该信号产生,则会被阻塞,直到上一个针对该信号的处理过程结束以后才会被递送
- 当进程正在执行类似更新数据库这样的敏感任务时,可能不希望被某些信号中断。这时可以通过信号掩码暂时屏蔽而非忽略掉这些信号,使其一旦产生即被阻塞于未决状态,待特定任务完成后,再回过头来处理这些信号

## sigprocmask

```c
int sigprocmask (int how, const sigset_t* sigset,sigset_t*  oldset);
// 功能:设置调用进程的信号掩码
// 参数:how:修改信号掩码的方式,可取以下值
// 			SIG_BLOCK - 将sigset中的信号加入当前信号掩码
// 			SIG UNBLOCK - 从当前信号掩码中删除sigset中的信号
// 			SIG_SETMASK - 把sigset设置成当前信号掩码
// 		sigset:信号集,取NULL则忽略此参数
// 		oldset:输出原信号掩码,取NULL则忽略此参数
// 	返回值:成功返回0,失败返回-1
```

## sigpending

```c
#include <signal.h>
int sigpending (sigset_t* sigset);
// 	功能:获取调用进程的未决信号集（正在屏蔽的一些信号）
// 	参数:sigset:输出未决信号集
// 	返回值:成功返回0,失败返回-1
```

# 可靠信号和不可靠信号的屏蔽

### 对不可靠信号的屏蔽

- 对于不可靠信号,通过sigprocmask函数设置信号掩码以后,每种被屏蔽信号中只有第一个会被阻塞,并在解除屏蔽后被递送,其余的则全部丢失

![信号集对不可靠信号的屏蔽](./images/信号集对不可靠信号的屏蔽.png)

### 对可靠信号的屏蔽

- 对于可靠信号,通过sigprocmask函数设置信号掩码以后,每种被屏蔽信号
  中的每个信号都会被阻塞,并按先后顺序排队,一旦解除屏蔽,这些信号会被
  依次递送

![信号集对可靠信号的屏蔽](./images/信号集对可靠信号的屏蔽.png)
