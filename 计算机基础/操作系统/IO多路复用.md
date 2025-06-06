# 1. 为什么需要IO多路复用

在讨论多路复用前，先介绍一下阻塞式IO和非阻塞式IO。

阻塞IO和非阻塞IO是两种不同的I/O处理方式，它们主要区别在于I/O操作是否会导致调用者等待。

**阻塞IO（Blocking IO）**

在阻塞IO模式下，当一个I/O操作（如读或写）发起时，如果数据还没有准备好（例如，等待数据从磁盘读取或从网络接收），则调用者（通常是一个线程或进程）会被阻塞，直到数据准备好为止。在此期间，调用者无法执行其他任务，只能等待I/O操作完成。

阻塞IO的优点是编程简单，容易实现。然而，它的缺点是当I/O操作耗时较长时，会导致调用者的低效率和资源浪费，尤其在高并发场景中。

**非阻塞IO（Non-blocking IO）**

在非阻塞IO模式下，当一个I/O操作发起时，如果数据还没有准备好，调用者不会被阻塞，而是立即返回一个错误码（例如，表示资源不可用）。调用者可以继续执行其他任务，然后在适当的时间点再次尝试I/O操作。

非阻塞IO的优点是可以提高调用者的效率和资源利用率，尤其适用于高并发场景。

阻塞IO和非阻塞IO的关键区分节点是数据准备阶段。



# 2. IO多路复用的技术

`select`, `poll`和`epoll`都是I/O多路复用技术，它们用于同时处理多个I/O操作，特别是在高并发网络编程中。

**select**

`select`是最早的I/O多路复用技术，它可以同时监视多个文件描述符（file descriptor, FD）的I/O状态（如可读、可写、异常等）。`select`函数使用一个文件描述符集合（通常是一个位图）来表示要监视的文件描述符，当有I/O事件发生时，`select`会返回对应的文件描述符集合。

select的主要限制如下：

* 文件描述符数量限制：`select`使用一个位图来表示文件描述符集合，这限制了它能够处理的文件描述符数量（通常是1024个）。
* 效率问题：当文件描述符数量较大时，`select`需要遍历整个文件描述符集合来查找就绪的文件描述符，这会导致较低的效率。
* 非实时性：每次调用`select`时，需要重新设置文件描述符集合，这会增加函数调用的开销。

**poll**

`poll`是为了克服`select`的限制而引入的一种I/O多路复用技术。`poll`使用一个文件描述符数组（通常是一个结构体数组）来表示要监视的文件描述符。与`select`类似，`poll`可以监视多个文件描述符的I/O状态。

poll的优点如下：

* 文件描述符数量不受限制：由于`poll`使用一个动态数组来表示文件描述符，因此它可以处理任意数量的文件描述符。
* 效率相对较高：`poll`在查找就绪的文件描述符时，只需要遍历实际使用的文件描述符数组，而不是整个文件描述符集合。

然而，`poll`仍然存在一些问题：

* 效率问题：尽管`poll`相对于`select`具有较高的效率，但当文件描述符数量很大时，它仍然需要遍历整个文件描述符数组。
* 非实时性：与`select`类似，每次调用`poll`时，需要重新设置文件描述符数组。

**epoll**

`epoll`是Linux特有的一种高效I/O多路复用技术，它克服了`select`和`poll`的主要限制。`epoll`使用一个事件驱动（event-driven）的方式来处理I/O操作，它只会返回就绪的文件描述符，而不是遍历整个文件描述符集合。

epoll的主要优点如下：

* 高效：`epoll`使用事件驱动的方式来处理I/O操作，因此它在处理大量文件描述符时具有很高的效率。当有I/O事件发生时，`epoll`可以立即得到通知，而无需遍历整个文件描述符集合。这使得`epoll`在高并发场景中具有更好的性能。
* 可扩展性：与`poll`类似，`epoll`可以处理任意数量的文件描述符，因为它使用一个动态数据结构来表示文件描述符。
* 实时性：`epoll`使用一个内核事件表来记录要监视的文件描述符和事件，因此在每次调用`epoll`时无需重新设置文件描述符集合。这可以减少函数调用的开销，并提高实时性。

`epoll`具有诸多优点，但它目前仅在Linux平台上可用。对于其他平台，可能需要使用类似的I/O多路复用技术，如BSD中的`kqueue`。

| 系统调用       | select                                                                              | poll                                                                               | epoll                                                           |
| ---------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| 事件集合       | 用户通过三个参数分别传入感兴趣的可读可写以及异常等事件，内核通过对这些参数的在线修改来反馈其中就绪的事件。如果用户需要的话需要创建三个fdset以监听不同类型的事件。 | 统一处理所有的事件类型，因此只需要一个事件集参数。用户通过pollfd.events来传入感兴趣的事件，内核通过修改pollfd.revents反馈其中就绪的事件。 | 内核通过一个事件表直接管理用户感兴趣的所有事件。每次调用epoll_wait，内核直接在调用参数的events中注册就绪事件。 |
| 应用程序索引效率   | 采用轮询方式，O(n)                                                                         | 采用轮询方式，O(n)                                                                        | 采用回调方式，O(1)                                                     |
| 最大支持文件描述符数 | 一般1024                                                                              | 65535                                                                              | 65535                                                           |
| 工作模式       | 条件触发                                                                                | 条件触发                                                                               | 条件触发或边缘触发                                                       |

## 边缘触发与条件触发分别是什么？

边缘触发（Edge-triggered）和条件触发（Level-triggered）是两种常见的事件触发方式，主要应用于I/O多路复用和中断处理等场景。

**边缘触发（Edge-triggered）**

边缘触发是指在事件状态发生变化的时刻触发一次，例如从无事件变为有事件。在I/O多路复用中，边缘触发意味着当某个文件描述符发生I/O事件（如变为可读或可写）时，我们只会收到一次通知。当收到通知后，我们需要处理该文件描述符上的所有数据，直到数据全部处理完毕，否则不会再收到通知。

边缘触发的优点是只在事件状态改变时触发，可以减少事件通知的次数。然而，边缘触发的缺点是我们需要确保在收到通知后处理所有相关数据，否则可能会遗漏某些事件。

**条件触发（Level-triggered）**

条件触发是指只要事件状态保持满足某种条件，就会持续触发。在I/O多路复用中，条件触发意味着只要某个文件描述符的I/O事件状态满足条件（如可读或可写），我们就会不断收到通知。

条件触发的优点是它可以确保我们不会遗漏任何事件，因为只要条件满足，就会持续触发。然而，条件触发的缺点是它可能导致大量的事件通知，从而增加处理开销。

# 3. select详解

假如能够预先传入一个socket列表，如果列表中的socket都没有数据，挂起进程，直到有一个socket收到数据，唤醒进程。这种方法很直接，也是select的设计思想。

为方便理解，我们先复习select的用法。在如下的代码中，先准备一个数组（下面代码中的fds），让fds存放着所有需要监视的socket。然后调用select，如果fds中的所有socket都没有数据，select会阻塞，直到有一个socket接收到数据，select返回，唤醒进程。用户可以遍历fds，通过FD_ISSET判断具体哪个socket收到数据，然后做出处理。

```c
int s = socket(AF_INET, SOCK_STREAM, 0);  
bind(s, ...)
listen(s, ...)

int fds[] =  存放需要监听的socket

while(1){
    int n = select(..., fds, ...)
    for(int i=0; i < fds.count; i++){
        if(FD_ISSET(fds[i], ...)){
            //fds[i]的数据处理
        }
    }
}
```

select的实现思路很直接。假如程序同时监视如下图的sock1、sock2和sock3三个socket，那么在调用select之后，操作系统把进程A分别加入这三个socket的等待队列中。

![](http://123.57.190.49:12121/api/image/RVL4BD04.png)

当任何一个socket收到数据后，中断程序将唤起进程。下图展示了sock2接收到了数据的处理流程。

ps：recv和select的中断回调可以设置成不同的内容。

![](http://123.57.190.49:12121/api/image/82F8NJFJ.png)

经由这些步骤，当进程A被唤醒后，它知道至少有一个socket接收了数据。程序只需遍历一遍socket列表，就可以得到就绪的socket。

![](http://123.57.190.49:12121/api/image/6N80N8L2.png)

这种简单方式行之有效，在几乎所有操作系统都有对应的实现。

# 4. epoll详解

## 4.1 select低效的原因

1. 将“维护等待队列”和“阻塞进程”两个步骤合二为一。每次调用select时，都需要执行系统调用，阻塞进程。

2. 程序不知道哪些socket收到数据，只能一个个遍历。如果内核维护一个“就绪列表”，引用收到数据的socket，就能避免遍历。

## 4.2 epoll的改进

epoll将这两个操作“维护等待队列”和“阻塞进程”分开，先用epoll_ctl维护等待队列，再调用epoll_wait阻塞进程。显而易见的，效率就能得到提升。

```c
int s = socket(AF_INET, SOCK_STREAM, 0);   
bind(s, ...)
listen(s, ...)

int epfd = epoll_create(...);
epoll_ctl(epfd, ...); //将所有需要监听的socket添加到epfd中

while(1){
    int n = epoll_wait(...)
    for(接收到数据的socket){
        //处理
    }
}
```

## 4.3 epoll的原理和流程

如下图所示，当某个进程调用epoll_create方法时，内核会创建一个eventpoll对象（也就是程序中epfd所代表的对象）。eventpoll对象也是文件系统中的一员，和socket一样，它也会有等待队列。 创建一个代表该epoll的eventpoll对象是必须的，因为内核要维护“就绪列表”等数据，“就绪列表”可以作为eventpoll的成员。

![](http://123.57.190.49:12121/api/image/FB62XR80.png)

创建epoll对象后，可以用epoll_ctl添加或删除所要监听的socket。以添加socket为例，如下图，如果通过epoll_ctl添加sock1、sock2和sock3的监视，内核会将eventpoll添加到这三个socket的等待队列中。当socket收到数据后，中断程序会操作eventpoll对象，而不是直接操作进程。

当socket收到数据后，中断程序会给eventpoll的“就绪列表”添加socket引用。如下图展示的是sock2和sock3收到数据后，中断程序让rdlist引用这两个socket。

![](http://123.57.190.49:12121/api/image/68844640.png)

eventpoll对象相当于是socket和进程之间的中介，socket的数据接收并不直接影响进程，而是通过改变eventpoll的就绪列表来改变进程状态。

当程序执行到epoll_wait时，如果rdlist已经引用了socket，那么epoll_wait直接返回，如果rdlist为空，阻塞进程。

假设计算机中正在运行进程A和进程B，在某时刻进程A运行到了epoll_wait语句，此时epoll为空。如下图所示，内核会将进程A放入eventpoll的等待队列中，阻塞进程。

![](http://123.57.190.49:12121/api/image/48Z2N840.png)

当socket接收到数据，中断程序一方面修改rdlist，另一方面唤醒eventpoll等待队列中的进程，进程A再次进入运行状态（如下图）。也因为rdlist的存在，进程A可以知道哪些socket发生了变化。

![](http://123.57.190.49:12121/api/image/TDZJ6D6D.png)




