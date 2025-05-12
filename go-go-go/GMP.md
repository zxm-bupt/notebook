# 1. 题记

毋庸置疑的，go的核心是goroutine，go的一切都是为了goroutine服务的，他引用自豪的channel也是为了goroutine能够能方便的通信。至于其他特性，只是为了方便代码和工程实现的语法糖罢了。

goroutine和less is more的原则，可以看成是go的一体两面，一个在实现上联系全局，一个在原则上掌控四方。

GMP是go实现并发的模型，也是goroutine的工作模型或者说调度模型。

# 2. 基本概念

* **G:** Goroutine

* **M：** Machine 内核级线程

* **P：** Processor 调度器

G-M-P由这三部分组成。虽然这个名字取的很抽象，我至今也不理解为什么kernal thread要命名为Machine，而scheduler命名为processor。

一个G执行的基本流程是：进入P的本地队列，等待P的调度，当P处理到当前的G时，将G放到一个M上执行。

# 3. 两个重要机制

G-M-P有两个重要的机制，一个时工作窃取，另一个是M的复用。

## 3.1 工作窃取

工作窃取指的是，当P的本地队列为空时，P当前绑定的M，会从其余P中窃取一半的G来执行。有这么几点需要注意，

1. 窃取是原子操作，要么窃取成功，要么窃取失败；

2. 窃取前，会等待一个随机数，发动窃取，以避免冲突。另外窃取的对象也是随机的；

3. 当窃取一个失败后，会继续尝试窃取其他P的G；

4. 窃取的P的范围是除了自己之外的所有P，直到窃取到G或者所有P窃取一遍都失败为止。

窃取通常窃取P的尾部的G。

当窃取后，仍然没有G在自己的队列中，则从全局队列中拿取少量的G。

为什么不先在全局队列中拿，一方面是为了保证每个P之间的负载均衡；另一方面是全局队列操作需要加全局锁，会带来额外的开销。

## 3.2 M的复用（hand off）

在讨论M的复用之前呢，我们需要搞明白为什么需要复用M。因为，M是内核级线程，创建内核级线程是需要进入内核态的，就会带来额外的开销。因此应该尽可能不要频繁创建M。

什么叫做复用，我们做这样一个假设，M上运行的G阻塞了，那M也需要跟着进行阻塞，这时候，P就可以从当前空闲的M中绑定一个其他M继续工作，如果没有其他M，则创建一个新的。当G阻塞结束，例如IO完成，那么G回到P的本地队列，M进入空闲队列。

值得一提，G会回到他开始绑定的P的本地队列，除非P的队列满了，他会进入全局队列。为什么优先进入原来P的本地队列，是根据局部性原理。一般的，processor与CPU的核心绑定，即一个CPU核心运行一个processor，这样可以极大的利用CPU并行计算的能力。而CPU多个核心之间不共享二级缓存和一级缓存，因此回到之前的P有利于更好的利用缓存。

# 4. 几种场景

## 4.1 场景1：

P 拥有 G1，M1 获取 P 后开始运行 G1，G1 使用 `go func()` 创建了 G2，为了局部性 G2 优先加入到 P1 的本地队列。

<img src="http://123.57.190.49:12121/api/image/6PZZ8BLJ.png" title="" alt="" data-align="center">

## 4.2 场景2：

G1 运行完成后 (函数：goexit)，M 上运行的 goroutine 切换为 G0，G0 负责调度时协程的切换（函数：schedule）。从 P 的本地队列取 G2，从 G0 切换到 G2，并开始运行 G2 (函数：execute)。实现了线程 M1 的复用。



<img src="http://123.57.190.49:12121/api/image/P24R8TFX.png" title="" alt="" data-align="center">

## 4.3 场景3：

假设每个 P 的本地队列只能存 3 个 G。G2 要创建了 6 个 G，前 3 个 G（G3, G4, G5）已经加入 p1 的本地队列，p1 本地队列满了。G2 在创建 G7 的时候，发现 P1 的本地队列已满，需要执行**负载均衡** (把 P1 中本地队列中前一半的 G，还有新创建 G **转移**到全局队列)，这些 G 被转移到全局队列时，会被打乱顺序。所以 G3,G4,G7 被转移到全局队列。

<img src="http://123.57.190.49:12121/api/image/F6LV2B2V.png" title="" alt="" data-align="center">

<img src="http://123.57.190.49:12121/api/image/0NF4R288.png" title="" alt="" data-align="center">

## 4.4 场景5

G2 创建 G8 时，P1 的本地队列未满，所以 G8 会被加入到 P1 的本地队列。

<img src="http://123.57.190.49:12121/api/image/T0FNB6RR.png" title="" alt="" data-align="center">

G8 加入到 P1 点本地队列的原因还是因为 P1 此时在与 M1 绑定，而 G2 此时是 M1 在执行。所以 G2 创建的新的 G 会优先放置到自己的 M 绑定的 P 上。

## 4.5 场景6

规定：**在创建 G 时，运行的 G 会尝试唤醒其他空闲的 P 和 M 组合去执行**

假定 G2 唤醒了 M2，M2 绑定了 P2，并运行 G0，但 P2 本地队列没有 G，M2 此时为自旋线程 **（没有 G 但为运行状态的线程，不断寻找 G）**。

M2 尝试从全局队列 (简称 “GQ”) 取一批 G 放到 P2 的本地队列（函数：`findrunnable()`）。M2 从全局队列取的 G 数量符合下面的公式：

$$
n = min(\frac{len(GQ)}{GOMAXPROCS} + 1, len(\frac{GQ}{2}))
$$

至少从全局队列取 1 个 g，但每次不要从全局队列移动太多的 g 到 p 本地队列，给其他 p 留点。这是**从全局队列到 P 本地队列的负载均衡**。

假定我们场景中一共有 4 个 P（GOMAXPROCS 设置为 4，那么我们允许最多就能用 4 个 P 来供 M 使用）。所以 M2 只从能从全局队列取 1 个 G（即 G3）移动 P2 本地队列，然后完成从 G0 到 G3 的切换，运行 G3。

<img src="http://123.57.190.49:12121/api/image/84P28BNH.png" title="" alt="" data-align="center">

<img src="http://123.57.190.49:12121/api/image/88D06D80.png" title="" alt="" data-align="center">

## 4.6 场景8

假设 G2 一直在 M1 上运行，经过 2 轮后，M2 已经把 G7、G4 从全局队列获取到了 P2 的本地队列并完成运行，全局队列和 P2 的本地队列都空了，如场景 8 图的左半部分。全局队列已经没有 G，那 m 就要执行 work stealing (偷取)：从其他有 G 的 P 哪里偷取一半 G 过来，放到自己的 P 本地队列。P2 从 P1 的本地队列尾部取一半的 G，本例中一半则只有 1 个 G8，放到 P2 的本地队列并执行。

<img title="" src="http://123.57.190.49:12121/api/image/08D66JTD.png" alt="" data-align="center">

## 4.7 场景9

G1 本地队列 G5、G6 已经被其他 M 偷走并运行完成，当前 M1 和 M2 分别在运行 G2 和 G8，M3 和 M4 没有 goroutine 可以运行，M3 和 M4 处于自旋状态，它们不断寻找 goroutine。为什么要让 m3 和 m4 自旋，自旋本质是在运行，线程在运行却没有执行 G，就变成了浪费 CPU. 为什么不销毁现场，来节约 CPU 资源。因为创建和销毁 CPU 也会浪费时间，我们希望当有新 goroutine 创建时，立刻能有 M 运行它，如果销毁再新建就增加了时延，降低了效率。当然也考虑了过多的自旋线程是浪费 CPU，所以系统中最多有 GOMAXPROCS 个自旋的线程 (当前例子中的 GOMAXPROCS=4，所以一共 4 个 P)，多余的没事做线程会让他们休眠。

## 4.8 场景10

 假定当前除了 M3 和 M4 为自旋线程，还有 M5 和 M6 为空闲的线程 (没有得到 P 的绑定，注意我们这里最多就只能够存在 4 个 P，所以 P 的数量应该永远是 M>=P, 大部分都是 M 在抢占需要运行的 P)，G8 创建了 G9，G8 进行了阻塞的系统调用，M2 和 P2 立即解绑，P2 会执行以下判断：如果 P2 本地队列有 G、全局队列有 G 或有空闲的 M，P2 都会立马唤醒 1 个 M 和它绑定，否则 P2 则会加入到空闲 P 列表，等待 M 来获取可用的 p。本场景中，P2 本地队列有 G9，可以和其他空闲的线程 M5 绑定。

<img src="http://123.57.190.49:12121/api/image/8J0262BR.png" title="" alt="" data-align="center">

值得注意：

1. 在GMP模型中，如果G（Goroutine）因为Go语言的互斥锁（go mutex）阻塞，M（Machine）不会阻塞。M会继续尝试从P（Processor）的队列中获取其他可运行的G，从而保证系统的整体并发能力

2. 如果M在执行G的过程发生**网络IO等操作阻塞**时（异步），阻塞G，**不会阻塞M**。M会寻找P中其它可执行的G继续执行，**G会被网络轮询器network poller 接手**，当阻塞的G恢复后，G1从network poller 被移回到P的 LRQ 中，重新进入可执行状态。异步情况下，通过调度，Go scheduler 成功地将 I/O 的任务转变成了 CPU 任务，或者说将内核级别的线程切换转变成了用户级别的 goroutine 切换，大大提高了效率。

## 4.9 场景11

G8 创建了 G9，假如 G8 进行了**非阻塞系统调用**。

​ M2 和 P2 会解绑，但 M2 会记住 P2，然后 G8 和 M2 进入系统调用状态。当 G8 和 M2 退出系统调用时，会尝试获取 P2，如果无法获取，则获取空闲的 P，如果依然没有，G8 会被记为可运行状态，并加入到全局队列，M2 因为没有 P 的绑定而变成休眠状态 (长时间休眠等待 GC 回收销毁)。

<img title="" src="http://123.57.190.49:12121/api/image/00Z0RTN8.png" alt="" data-align="center">

# 5. 抢占式调度

* Go 1.2 中实现了**基于协作的“抢占式”调度**  
  协作式：是否让出p的决定权在groutine自身。  

* 编译器会在调用函数前插入 [runtime.morestack](https://zhida.zhihu.com/search?content_id=225335612&content_type=Article&match_order=1&q=runtime.morestack&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDcyNDI2MjEsInEiOiJydW50aW1lLm1vcmVzdGFjayIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjIyNTMzNTYxMiwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.M1HWw77KQRGWuquvBcDWrgzsLBv1HjnVkQ-FdVuhHNw&zhida_source=entity)，让运行时有机会在这段代码中检查是否需要执行抢占调度

* Go语言运行时会在垃圾回收暂停程序、系统监控发现 Goroutine 运行超过 10ms，那么会在这个**协程设置一个抢占标记**

* 当**发生函数调用时**，可能会执行编译器插入的 runtime.morestack，它调用的 runtime.newstack会**检查抢占标记**，如果有抢占标记就会触发抢占让出cpu，切到调度主协程里

这种解决方案只能说局部解决了“饿死”问题，只在有函数调用的地方才能插入“抢占”代码（埋点），**对于没有函数调用而是纯算法循环计算的 G，Go 调度器依然无法抢占**。  

* Go 1.14 中实现了**基于信号的“抢占式”调度**  
  不管协程有没有意愿主动让出 cpu 运行权，只要某个协程执行时间过长，就会发送信号强行夺取 cpu 运行权。  

* M 注册一个 [SIGURG](https://zhida.zhihu.com/search?content_id=225335612&content_type=Article&match_order=1&q=SIGURG&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NDcyNDI2MjEsInEiOiJTSUdVUkciLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyMjUzMzU2MTIsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.0EQhdyM-6ddWXDLo6k4Jp5KGZuE2IHP4jCcQXJCfi6k&zhida_source=entity)**信号的处理函数**：sighandler

* sysmon启动后会间隔性的进行监控，最长间隔10ms，最短间隔20us。如果**发现某协程独占P超过10ms，会给M发送抢占信号**

* M 收到信号后，内核执行 sighandler 函数把当前协程的状态从_Grunning正在执行改成 _Grunnable可执行，**把抢占的协程放到全局队列里**，M继续寻找其他 goroutine 来运行

* 被抢占的 G 再次调度过来执行时，会继续原来的执行流

# 5. 总结

显然Go的GMP实现并不负责，好吧，也很复杂，但是总比JRE这种Java运行时要简单多。go真实把less is more贯彻到底，很难想象最初的版本，没有P，没有抢占式调度，究竟是怎么运行的。
