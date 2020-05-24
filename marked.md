# 日常一些知识，遇到的BUG，听到的技术等记录

### Q：为什么Rpckit框架下产生的core文件，有时候是进程名，有时候是一个rc_thw_xxx的名字？

- 曙光提的问题，看了下  `/proc/sys/kernel/core_pattern`显示为 `/data/coredump/core-%e-%p-%t`，即进程名，进程号，时间戳
- 框架内部使用 `pthread_setname_np` 设置了线程名，在子线程内产生core，如果设置了线程名（子线程默认继承主线程名字，也就是进程名，这里精确的表述应该是复写了自己的线程名）core文件是子线程的名字
- Rpckit里面搜索`rc_thw`能发现问题，`RcThreadHandlerRoutine`函数设置的。另一种方式是通过`prctl(PR_SET_NAME, thread_name)`来设置
- 测试代码：https://github.com/xfmeng17/tempwork/blob/master/cpp/test/threadcore.cc

### Q：git如何批量修改提交的user和email

- https://www.cnblogs.com/se7end/p/12704976.html

### Q：















