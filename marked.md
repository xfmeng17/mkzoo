# 日常一些知识，遇到的BUG，听到的技术等记录

### 2021-02-23 KFIFO再理解

> https://www.linuxidc.com/Linux/2016-12/137936.htm

![kfifo](https://github.com/xfmeng17/mkzoo/blob/master/picture/2021_02/kfifo.jpg)

### 2021-01-06 从gettimeofday到内存屏障

起因是，使用gettimeofday()减去trpc::context->GetRecvTimestamp()属性上报溢出（已解决，存在uint64_t溢出问题）

去读trpc::time.cc的代码，读了一串知识盲区：

1. trpc::time.cc trpc_rdtsc \__asm__ __volatile__("rdtsc" : "=a"(low), "=d"(high))

2. 为啥实现上用rdtsc，是为了防止在就机器上，gettimeofday性能不行：[浅谈时间函数gettimeofday的成本](https://blog.csdn.net/russell_tao/article/details/7185588)

3. 软硬终端，软中断是啥？[点个外卖，我把「软中断」搞懂了](https://www.mdeditor.tw/pl/ggsn)

4. > 不过，软中断不只是包括硬件设备中断处理程序的下半部，一些内核自定义事件也属于软中断，比如内核调度等、RCU 锁（内核里常用的一种锁）等。

   RCU 锁 是啥来的？[深入理解 Linux 的 RCU 机制](https://zhuanlan.zhihu.com/p/30583695) 读到了 smp_wmb() 和内存屏障

5. 内存屏障到底是啥来的？[Memory barrier是什么？](https://www.zhihu.com/question/20228202) 这个了解了，又读了 [理解 Memory barrier（内存屏障）](ttps://blog.csdn.net/zhangxiao93/article/details/42966279) 陷入到这个文章中

6. 内核kfifo的实现，源码使用了smp_mb() smp_rmb() smp_wmb()，问了问丙玉

7. 还没看 https://github.com/apache/incubator-brpc/blob/master/docs/cn/atomic_instructions.md

补充:

1. `sar -n DEV 1 3` 查看网卡，参数意义：https://blog.51cto.com/461205160/1939549
2. 查看每个软中断类型的中断次数的变化速率：`watch -d cat /proc/softirqs`

------

### Q：为什么Rpckit框架下产生的core文件，有时候是进程名，有时候是一个rc_thw_xxx的名字？

- 曙光提的问题，看了下  `/proc/sys/kernel/core_pattern`显示为 `/data/coredump/core-%e-%p-%t`，即进程名，进程号，时间戳
- 框架内部使用 `pthread_setname_np` 设置了线程名，在子线程内产生core，如果设置了线程名（子线程默认继承主线程名字，也就是进程名，这里精确的表述应该是复写了自己的线程名）core文件是子线程的名字
- Rpckit里面搜索`rc_thw`能发现问题，`RcThreadHandlerRoutine`函数设置的。另一种方式是通过`prctl(PR_SET_NAME, thread_name)`来设置
- 测试代码：https://github.com/xfmeng17/tempwork/blob/master/cpp/test/threadcore.cc

### Q：git如何批量修改提交的user和email

- https://www.cnblogs.com/se7end/p/12704976.html

















