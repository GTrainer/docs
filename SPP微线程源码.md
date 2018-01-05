## SPP微线程源码学习笔记


### 1 目录
**SPP\_proj/trunk/sync\_frame/micro\_thread**


### 2 源码列表
- Makefile
- hash_list.h
- heap.h
- heap_timer.h
- heap_timer.cpp
- mt_version.h
- mt_msg.h
- mt\_mbuf\_pool.h
- mt\_mbuf\_pool.cpp
- mt\_sys\_hook.h
- mt\_sys\_hook.cpp
- epoll_proxy.h
- epoll_proxy.cpp
- micro_thread.h
- micro_thread.cpp
- mt_session.h
- mt_session.cpp


### 3 源码详解

#### 3.1 hash_list.h
实现了一个开链式的哈希表，主要包括两个结构体：

- HashKey，哈希节点的结构体，链表结构。
- HashList，哈希列表结构体，默认取小于100000的最小的质数作为开链的个数。

#### 3.2 heap.h
实现了一个自定义规则的最小堆，主要包括两个数据结构：

- HeapEntry，堆节点结构体。
- HeapList，堆列表结构体，默认取100000个节点。

#### 3.3 heap\_timer.h->heap\_timer.cpp
最小堆基础上实现的定时器管理类，主要包括两个数据结构：

- CTimerNotify，单个定时器对象，继承自HeapEntry结构体。
- CTimerMng，定时器管理器，包含一个HeapList结构体，管理和检测是否有定时器到期。

#### 3.4 mt_version.h
定义当前库的版本号，一个弱符号字符数组，如果和其他文件的定义冲突，则被覆盖。

#### 3.5 mt_msg.h
微线程处理基类，所有通过微线程处理的函数都通过继承该类实现。

#### 3.6 mt\_mbuf\_pool.h->mt\_mbuf\_pool.cpp
微线程消息缓存池，用TAILQ链表来管理的接收和发送的消息队列。

**三种消息池类型：**

- 未知类型
- 接收类型
- 发送类型

**三个结构体：**

- MtMsgBuf，单个消息的缓冲区，实际上是一个双链表的节点。
- MsgBufMap，继承自HashKey，实际上是一个哈希节点，同时管理一个MtMsgBuf结构的双链表，根据存储的缓冲区大小作为哈希值。
- MsgBuffPool，包含了一个HashList结构体，实际上是一个哈希列表，单例模式实现，用于管理全局缓冲区的分配和回收，每一条链实际只有一个哈希节点，节点内部用TAILQ管理内存的分配和释放。

#### 3.7 mt\_sys\_hook.h->mt\_sys\_hook.cpp
采用钩子的形式接管系统的socket api。

- g\_mt\_hook\_flag全局控制标志，用来控制是采用系统同步、异步抑或微线程同步的方式来通信。
- 是否将套接字的O_NONBLOCK标志置位决定是采用系统的异步实现还是微线程版本的异步实现。

#### 3.8 epoll\_proxy.h->epoll\_proxy.cpp
封装EPOLL对象，管理网络IO事件，主要包含三个结构体：

- EpollerObj，管理单个网络文件描述符需要监听的网络事件：
	- InputNotify，当文件描述符可读时调用。
	- OutputNotify，当文件描述符可写时调用。
	- HangupNotify，当文件描述符出错时调用。
- FdRef，管理一个文件描述符被引用的次数。
- EpollProxy，Epoll管理代理，管理全局唯一的的Epoll文件描述符：
	- getrlimit、setrlimit，改变系统对软硬件资源的限制。
	- EpollDispath，分发监听到的网络事件。

#### 3.9 micro\_thread.h->micro\_thread.cpp
封装微线程实现，包括对微线程的调度管理，主要包括一下数据结构：

- MtStack，线程栈，为微线程分配的私有栈空间。
- Thread，线程基类，主要包含线程的基本操作以及微线程的上下文切换：
	- 包含一个MtStack对象，表示自己的私有栈空间。
- ScheduleObj，全局调度管理器，单例实现，与MtFrame联合使用，进行全局线程的调度。
- MicroThread，微线程类，继承自Thread，包含线程的当前状态、所处的状态队列。每个线程还包含一个子线程队列，可以支持一组线程程并发执行。疑问：关注一组文件对象的作用？
- LogAdapter，记录日志。
- ThreadPool，线程池，管理线程。负责分配微线程栈空间的大小以及线程的创建和销毁。
- MtFrame，微线程框架，单例实现，负责整个微线程的调度。

#### 3.10 mt\_session.h->mt\_session.cpp
每个连接对应一个会话状态，该文件封装了对会话状态的管理。主要包括下面两个数据结构：

- ISession，单个会话状态接口，一个连接对应一个会话状态。继承自HashKey，实际上是一个哈希节点。
- SessionMgr，会话状态管理类，单例实现，管理全局的会话状态。包含了一个HashList，实际上是一个哈希表。
