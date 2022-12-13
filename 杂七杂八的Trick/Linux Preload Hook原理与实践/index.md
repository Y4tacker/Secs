# Linux Preload Hook原理与实践

## 转载

和以前一样好文章有时候莫名奇妙被删除了备份下

http://events.jianshu.io/p/f78b16bd8905

## Preload简介

### Linux常见Hook技术对比

| 技术类型     | 生效范围     | 生效时机       | 依赖注入 | 层级 | 安全性 | 稳定性 | 开发运维难度 |
| ------------ | ------------ | -------------- | -------- | ---- | ------ | ------ | ------------ |
| 内核模块     | 所有进程     | 加载内核模块后 | 否       | 低   | 最高   | 中     | 极高         |
| Inline Hook  | 当前进程     | hook后         | 是       | 中   | 中     | 中     | 高           |
| Got Hook     | 当前进程模块 | hook后         | 是       | 中   | 低     | 良     | 中           |
| Preload Hook | 所有进程     | hook后         | 否       | 中   | 低     | 优     | 低           |
| 系统文件修改 | 所有进程     | 文件修改后     | 否       | 中   | 高     | 中     | 高           |

<div style="padding-left:300px;">表1 Linux常见Hook技术对比</div>

表1注：
 ① 依赖注入，代表在使用该技术hook第三方进程前，是否需要先进行注入
 ② 层级，代表hook点在整个系统调用的位置，越底层则层级越低
 ③ 稳定性越低，代表要通过这种技术做到稳定所需要的技术和时间成本较高
 ④ 安全性，代表该项技术被绕过的难易程度

### 函数调用类型

  按函数调用在整个系统所处层级划分，可分为系统调用和库函数调用：

- 系统调用
    系统调用是指，真正执行逻辑实现于系统内核的库函数，通常是操作系统资源相关的例如文件、内存、进程、线程、网络等函数，如open函数。对这类函数的Hook可以做到较高安全性。`/usr/include/bits/syscall.h`头文件定义了当前系统支持的所有系统调用
- 库函数
    真正逻辑实现于用户态动态库的函数，除系统调用以外的库函数都算该类，如memcpy函数

  按函数调用时机划分，可分为静态调用和动态调用

- 静态调用
    静态调用是指，直接使用函数调用运算符("()")调用自身或外部库导出函数
- 动态调用
    动态调用是指，使用dlopen/dlsym等方式直接获取到函数指针，进行调用的方式

### 内核模块Hook

  通常是从内核源码特殊位置，修改回调、修改中断表；或修改重编译内核，导出内部函数，从而跳转到自定义函数，开发内核模块实现hook。

- 可以拦截到所有应用层系统调用，应用层无法绕过
- 开发调试复杂，测试周期长，升级和卸载内核模块带来稳定性问题

### 应用层Inline Hook

  应用层内联hook，即直接修改二进制函数体的汇编指令，修改执行逻辑使其跳转到自定义函数，开发应用层模块实现hook。

- 可以拦截到系统调用和普通库函数
- 由于linux系统本身具有多个发行版本及指令集，不容易做到通用
- 可以通过自定义实现底层函数或恢复模块内存方式绕过

### Got表

  当在程序中引用某个共享库中的符号时，编译链接阶段并不知道这个符号的具体位置，只有等到动态链接器将所需要的共享库加载时进内存后，符号的地址才会最终确定。因此，需要有一个数据结构来保存符号的绝对地址。编译器在编译C/C++程序时，会根据代码中对外部库函数的静态调用情况构造成一个符号索引表，使代码中外部库函数的调用暂时指向每个索引，在程序运行时进程加载器([进程加载器的概念](#进程加载器的概念))会将每个索引表项指向真正的函数地址，从而完成外部库函数调用。这种方式实现了在编译期时处理(动态加载模块的)函数调用问题。
  在Windows系统中，可执行程序为PE结构，该表为`import table`(俗称导入表)，对应`.idata`段；Linux/Android系统中，可执行程序为ELF结构，该表即为got(Global Offset Table)表，对应`.got`段；MacOS/iOS系统中，可执行程序为MachO格式，该表对应为`__nl_symbol_ptr`和`__la_symbol_ptr`段。在本文中统称导入表。

### 应用层Got Hook

  应用层got表hook，即在运行阶段修改程序本身got表，这样调用api的逻辑，就会相应的跳转到用户自定义函数中。

- 可以拦截系统调用和普通库函数
- 由于只需要考虑ELF格式因此实现难度较为简单
- 可以通过自定义实现底层函数或恢复got表内存方式绕过

### 应用层Preload Hook

  对于Preload的解释，详见[Preload Hook原理](#Preload Hook原理)。Preload Hook是指利用系统支持的preload能力，将模块自动注入进程实现hook。可以通过以下手段使用Preload技术：一种是环境变量配置(LD_PRELOAD)；另一种是文件配置：(/etc/ld.so.preload)。

- 若使用命令行指定LD_PRELOAD则只影响该新进程及子进程；若写入全局环境变量则LD_PRELOAD对所有新进程生效；父进程可以控制子进程的环境变量从而取消preload
- 文件preload方式影响所有新进程且无法被取消
- 可以拦截到系统调用和普通库函数
- 实现和操作最为简单，只需要编写同名系统调用函数即可实现hook
- 可以使用动态调用方式或自定义实现方式绕过

## Preload Hook原理

### 进程模块表的链状结构

  下面是从源码中摘录的部分定义，可以看到常见系统均(至少)使用单链表或双链表方式存储模块表，甚至MacOS和Windows操作系统也是采用类似方式。使用链表方式存储，便于增删查改(一般程序加载模块数在百级别以内)。



```c
// from glibc/elf/link.h                for Linux
struct link_map     
{
    /* These first few members are part of the protocol with the debugger.
       This is the same format used in SVR4.  */
    ElfW(Addr) l_addr;        /* Difference between the address in the ELF
                   file and the addresses in memory.  */
    char *l_name;        /* Absolute file name object was found in.  */
    ElfW(Dyn) *l_ld;        /* Dynamic section of the shared object.  */
    struct link_map *l_next, *l_prev; /* 这里为链表结构.  */
};  
struct link_map _dl_loaded; // 全局模块表头
```



```cpp
// from bionic/linker/linker_soinfo.h            for Android 8
struct soinfo {
public:
      const ElfW(Phdr)* phdr;
      size_t phnum;
      ElfW(Addr) base;
      size_t size;
      ElfW(Dyn)* dynamic;
      soinfo* next;  /* 这里为链表结构  */
      ....;
};
struct soinfo solist; // 全局模块表头
```



```cpp
// from bionic/linker/linker.h                for Android 4-7
struct soinfo
{
    const char name[SOINFO_NAME_LEN];
    Elf32_Phdr *phdr;
    int phnum;
    unsigned entry;
    unsigned base;
    unsigned size;
    int unused;  // DO NOT USE, maintained for compatibility.
    unsigned *dynamic;
    unsigned wrprotect_start;
    unsigned wrprotect_end;
    soinfo *next;
    ...
};  
struct soinfo solist; // 全局模块表头
```



```cpp
// from dyld/src/ImageLoader.h                    for Macos/iOS
class ImageLoader {
public:
    typedef uint32_t DefinitionFlags;
    static const DefinitionFlags kNoDefinitionOptions = 0;
    static const DefinitionFlags kWeakDefinition = 1;
    ......
    struct DynamicReference {                    /* 这里为链表结构 */
      ImageLoader* from;                            
      ImageLoader* to;
    };
    ......
};
static std::vector<ImageLoader*>    sAllImages; // 全局模块表数组
```



```cpp
// from wrk/public/sdk/inc/ntpsapi.h            for Windows
typedef struct _PEB_LDR_DATA {
    ULONG Length;
    BOOLEAN Initialized;
    HANDLE SsHandle;
    LIST_ENTRY InLoadOrderModuleList;            /* 这里为链表结构 */
    LIST_ENTRY InMemoryOrderModuleList;        /* 这里为链表结构 */
    LIST_ENTRY InInitializationOrderModuleList;    /* 这里为链表结构 */
    PVOID EntryInProgress;
} PEB_LDR_DATA, *PPEB_LDR_DATA;
// NtCurrentPeb()->Ldr       全局模块表头
```

### Linux系统Preload Hook

#### 进程加载器的概念

  先来介绍几个名词

- 进程加载器
    属于操作系统的一部分，用于加载程序和动态库。加载器是执行程序和代码必不可少的组件，是一种用户态的模块，用户在shell中启动进程后，首先加载的是进程加载器模块，它负责将进程文件从文件形态映射到内存中、加载相应的依赖模块、初始化可执行模块、初始化进程所需的环境，最后调用程序入口点(main)执行真正的工作。加载器在模块表中的位置总是最前。下表是常见系统的进程加载器：

| Bit  | Linux                       | Android              | MacOS/iOS     | Windows                         |
| ---- | --------------------------- | -------------------- | ------------- | ------------------------------- |
| 64   | /lib64/ld-linux-x86-64.so.2 | /system/bin/linker64 | /usr/lib/dyld | %SYSTEMROOT%\system32\ntdll.dll |
| 32   | /lib/ld-linux.so.2          | /system/bin/linker   | /usr/lib/dyld | %SYSTEMROOT%\system32\ntdll.dll |

<div style="padding-left:300px;">表2 多个平台的进程加载器</div>

- Preload技术
    Preload技术是Linux系统自身支持的模块预加载技术，这意味着进程加载器在加载进程时，会在模块表中首先插入指定的预加载模块，然后再插入进程所依赖的其他模块，预加载模块在模块表的位置总是位于加载器之后。Linux系统中，ELF格式的导入表只存储了符号(包括导出的全局对象、全局变量以及全局函数)名，因此在进程加载器初始化外部符号时，从模块链表头开始按模块搜索直到遇到该符号名。这点不同于Windows的PE格式，后者的导入表同时存储了符号名以及对应的模块名，这样进程加载器在初始化外部符号时已经确知需要加载哪个模块，若不存在则直接报错异常退出。ELF进程加载器这样的搜索顺序便会影响调用库函数时产生的符号查找的结果，具体会影响下面几点：
- - 静态调用的函数
      这类函数出现于编译得到二进制文件的导入表中，在被进程加载器映射到内存并初始化相关结构之后，导入表会被真实函数地址填充，而进程加载器正是通过模块链为导入表分配地址。假设两个包含相同符号/函数名的模块分别先后加载，那么先加载的处于链表靠前位置，进而会被进程加载器选中而当做真实函数地址。
- - 动态调用的函数，且不指定模块
      如果dlsym函数第一个参数被指定为RTLD_DEFAULT时，dlsym函数会匹配模块链中第一个包含该符号的模块，进而会被进程加载器选中而当做真实符号地址；若第一个参数指定的是RTLD_NEXT，dlsym函数会按模块链搜索下一个包含该符号的模块，并将该地址选为符号地址。(注：导出符号为导出函数或导出变量或常量)

  如前述，Linux/Android系统支持环境变量和文件方式的Preload；MacOS/iOS系统则提供了环境变量`DYLD_INSERT_LIBRARIES`用于Preload，同时提供`__interpose`结构用于Hook；Windows系统未提供环境变量方式的Preload，而提供了注册表方式注入。

#### Preload特点

  Preload技术有自身的作用范围，如果将Preload环境变量到全局环境(/etc/profile)或使用Preload文件时，Preload便在所有在此(配置)时间节点之后运行的进程生效，而无法影响已经在运行的进程；若将Preload环境变量作为启动进程的环境变量(如shell中执行`LD_PREOAD=/lib64/inject.so ./myprocess`)，则Preload只对当前进程生效。默认情况下进程创建的子进程也会继承Preload环境变量，而父进程可以在子进程初始化前修改子进程的环境变量，从而避免往子代传播。Preload具体使用方式如下：

- LD_PRELOAD环境变量  指定动态库全路径，触发时机：启动时
- /etc/ld.so.preload  指定动态库全路径，触发时机：启动时

  进程在启动后，会按照一定顺序加载动态库：

- 加载环境变量LD_PRELOAD指定的动态库
- 加载文件/etc/ld.so.preload指定的动态库
- 搜索环境变量LD_LIBRARY_PATH指定的动态库搜索路径
- 搜索路径/lib64下的动态库文件

  如前述，模块通过Preload技术添加到模块链后，进程加载器在解析导入表时，会按照链表顺序依次查找符号，因此如果在Preload动态库中实现并导出了同名(而不是C++修饰名)库函数，即可实现Hook。下面是一个简单例子，捕获所有socket函数：



```c
// inject.c
#include <stdio.h>
#include <sys/socket.h>
 __attribute__ ((visibility("default"))) int socket(int family, int type, int protocol) {
    printf("detect socket call\n");
    return -1;
}
__attribute__((constructor)) void main() {
    printf("module inject success\n");
}
/* 
    gcc inject.c --shared -fPIC -o inject.so
    LD_PRELOAD=$PWD/inject.so ping 127.0.0.1
    module inject success
    detect socket call
*/
```

## 应用Preload Hook存在的问题和解决方案

### Preload是否能作用于动态加载(dlopen/dlsym)的函数？

  这里和静态引用函数相比较，如前述，若preload模块已经导出同名函数，进程启动时需要借助进程加载器处理静态引用函数，从模块链中按模块加载顺序查找，直到遇到匹配的函数指针，这个函数此时位于preload模块中。而在动态调用过程中，指定动态库名及符号名的动态调用会在查找时匹配动态库名，再在该动态库符号表中查找符号名，因此会返回原始函数指针。

### Preload是否能作用于子进程(fork/vfork/clone)(execve/system/popen/...)？

  由于preload是进程加载器的行为，因此如果能触发进程加载器就能触发preload。对于第一类函数(fork/vfork/clone)，由于没有对进程进行重新初始化的操作，而仅仅是对进程空间的复制行为，因此不能触发preload；对于第二类函数(execve/system/popen/...)，其首先要调用fork产生一个子进程，然后需要使用新的可执行文件对子进程的进程空间重新初始化，这个过程势必会触发preload过程。

  另外一个重要的考虑因素是，Preload只在部署之后对新进程生效，因此需要作为一个独立服务的可启动项，考虑系统其他服务启动顺序和时机。

### 对Docker / Chroot的支持？

  Paas容器技术提供了和宿主隔离的独立环境，在这种环境下文件系统是相互隔离的，而Preload技术则基于文件系统(ld.so.preload)，因此不存在一个绝对完美的方法提供对容器类场景的支持。下面提供一些特殊方式来解决该需求，对于Docker 分2种情况讨论：

- 若容器提供了文件映射机制(如docker run -v)，在这种情况下至少要求映射/etc/ld.so.preload及Preload动态库以实现基本的Preload机制，而这本身又需要宿主系统和容器系统相近，保证系统稳定性。
- 若容器系统和宿主系统相差较大，则需要考虑在容器系统内全新部署Preload相关组件。

  对于类似于使用chroot机制进行文件隔离的场景，需要针对性的修改chroot环境中对应的Preload配置文件及部署拦截模块进行Preload。

### 如何拦截动态加载的符号？

  如前述，Preload是进程加载器的行为，因此对于静态引用的符号生效；这里暂且将动态调用限制在仅使用dlopen/dlsym函数上，对于明确指定了动态库路径和符号名的dlopen/dlsym的动态调用，必然要对dlsym本身进行hook，这样引发的问题是怎样得到原始的dlopen/dlsym及其他符号名，因为通常我们会在hook函数执行前/后，执行一次原始函数，而这个原始函数指针按常规方式需要由dlsym来获取。如果静态使用dlopen/dlsym则会调用hook模块中的自定义dlopen/dlsym中去，从而产生死循环。要解开这个循环必须要使用更底层的方式来做。一种选择是手动解析/proc/pid/map中的模块导出表，从而获取函数基址。更通用的方式则是使用GNU提供的dl_iterate_phdr，此函数存在于进程加载器linker中。所以一个典型的preload hook过程如下：

- 使用dl_iterate_phdr迭代出所有需要进行hook的函数地址，保存为原始函数
- 实现同名函数，在执行hook逻辑前/后，执行原始函数调用
- 在自定义dlopen/dlsym，若匹配到hook函数名，则返回hook模块本身的同名函数，以实现hook动态调用函数的目的

### 如何更安全的部署preload模块？

  由于preload函数会影响所有新启动的进程，若preload模块存在缺陷，最差的情况是无法执行任何命令(execve/system/popen/...)；此种情况下若进行重启，重启所创建的任何进程均会无法正确执行而导致系统无法使用，从而陷入死循环，这种情况极难处理。这里提供的解决方式是使用mount命令，其特点是，在删除源文件或重启后失效，利用这个特点，首先将ld.preload.so拷贝为ld.preload.so.new，然后使用mount将ld.preload.so.new绑定到ld.preload.so上。此时若preload模块存在缺陷，只需要重启即可去除preload。

## Preload Hook应用之——网络访问日志

  由前述可知，Preload技术并不是一种安全的技术，此技术多用于非安全应用的API拦截、日志记录、流量监控或打补丁等场景。在Preload技术上层即使构造比较复杂的系统，如果该系统和Preload本身是串联的，这种情况同样也是非安全，因此这里我们只讨论非安全对抗的应用模型，以网络通信监控为例，具体应用需要视具体业务而定。

#### 策略在so中

  如上图所示，每个主机的代理进程(Agent)负责维护策略，通过RPC从远程服务器获取监控策略，并下发给每个被管理(通过Preload Hook注入inject.so)的进程(Client)。这些进程加载和解析由代理进程下发的策略，在触发网络通信时在inject.so中对五元组(协议/源IP/源PORT/目的IP/目的PORT)自行做判决，这种模型存在以下特点：

- 所有Client负责加载和解析策略，存在一定内存开销；另外Client进程容易泄露策略数据
- 无额外网络/IPC开销
- Agent不存在、被杀死或者崩溃时，不影响Client中inject.so的拦截策略
- Preload只影响新启动的进程，因此策略的更新&运维较为困难

#### 策略在代理进程中

  如上图所示，每个主机的代理进程负责维护策略，同时对被管理的进程发送的五元组数据进行判决和响应，这些Client在触发网络通信时会通过inject.so向代理进程发送IPC五元组数据，根据结果选择拦截或允许。这种模型存在以下特点：

- 每个主机的Agent负责加载和解析策略
- 所有Client需要进行IPC通信，存在一定资源开销
- Agent不存在、被杀死或者崩溃时，会影响主机中每个Client中inject.so的拦截策略
- Preload只影响新启动的进程，因此策略的更新只需要向server下发策略即可

#### 策略在远程主机上

  如上图所示，和前一个模型不同的是，每个主机的代理进程通过RPC转发判决请求到服务器并返回结果。这种模型存在以下特点：

- 每次触发网络访问时都需要进行一次或多次RPC和IPC访问，对网络性能和主机性能要求较高
- Agent不存在、被杀死或者崩溃时，会影响主机中每个Client中inject.so的拦截策略
- Server宕机或被攻击时，会影响主机所对应所有Client中inject.so的拦截策略
- Preload只影响新启动的进程，因此策略的更新只需要向server下发策略即可

### 拦截函数

  需要拦截的点如下：

- dlopen/dlsym                拦截加载libc.so.6的行为，以便挂钩网络函数
- accept/accept4/connect    拦截TCP连入/连出请求
- send/recv系                拦截UDP单个连接请求(TCP/UDP通用)，包括send/recv/sendto/recvfrom/sendmsg/recvmsg/write/read

  在上述函数中，需要获取三元组标识一次通信过程，包括(协议，本地端，远端)，本地端和远端均由一个地址和一个端口构成。获取这些信息的方式一般如下：

- 判断文件描述符fd是否为套接字。对于send/recv系的read/write，fd可能是普通文件；另外忽略unix socket
- 获取协议类型。协议族(family:IPv4/IPv6)和协议类型(socktype:TCP/UDP)可以通过getsockname/getsockopt方式获取。
- 获取本地端。通过getsockname获取本地绑定的端口和地址，若套接字未使用bind函数绑定，则使用默认地址和随机端口通信。适用于服务端
- 获取远端。(in/out指代数据流方向)通过accept(in)/accept4(in)/connect(out)获取tcp远端，通过recvfrom(in)/recvmsg(in)/sendto(out)/sendmsg(out)获取udp远端。对于send(out)/write(out)/recv(in)/read(in)和未指定远端的sendto(out)/sendmsg(out)的套接字，会使用经由connect函数设置的默认远端进行通信，可以通过getpeername获取。

## 总结

  本篇文章详细介绍了Preload原理、使用方式及适用场景，最后通过网络策略拦截案例来例证该技术的适用场景。希望对大家有帮助。

## 参考文档

man手册
