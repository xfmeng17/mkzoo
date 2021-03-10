# TODODO

### 2021-03

1. 零拷贝

### 2020-09

1. ObjectPool 使用，复用fd
   - **2020-10-27 15：30** https://github.com/xfmeng17/tempwork/tree/master/cpp/fdreuse 对象池并不适用，不能解决淘汰过期FD的问题
2. PB 反射及PB相关：https://izualzhy.cn/protobuf-message-reflection
   - **2020-09-13 13：30** PB反射 https://github.com/xfmeng17/tempwork/commit/f67070e323c6b5281cde3d34d348b254d9c3b30b
   - **2020-09-13 15：30** PB编码 https://izualzhy.cn/protobuf-encoding 其实就是翻译自google文档https://developers.google.com/protocol-buffers/docs/encoding
3. trpc 请求decode失败后直接跳到encode rsp，为啥
   - **2020-10-27 15：30** 因为pb里面使用了string然后实际传的是bytes，导致pb内部的utf8 check逻辑失败，导致trpc框架decode失败，应该不是直接跳转到encode rsp，这里还需要进一步确认
4. golang 规则表达式，能否C++？https://github.com/Knetic/govaluate
5. TdBank SDK 为啥注释调-Wl 就能make了？

### 2020-08

1. boost线程池生产者消费者，ESReport可以实战下：https://git.code.oa.com/yoo_rec/RecSys/reviews/205/diffs
   	- 2020-08-16
   - https://github.com/xfmeng17/tempwork/blob/master/cpp/boost/lockfree_demo.cc
   - https://github.com/xfmeng17/tempwork/blob/master/cpp/boost/pcdemo.cc

### 2020-06

1. 递归mutex的实现https://blog.csdn.net/mcc12356/article/details/106947074

2. [Lock Free中的Hazard Pointer](http://www.yebangyu.org/blog/2015/12/10/introduction-to-hazard-pointer/)

3. [实现无锁的栈与队列(5)：Hazard Pointer](https://www.cnblogs.com/catch/p/5129586.html)

4. 内部小哥**carelye**的无锁hashmap

5. [分布式存储系统设计（3）—— 存储结构](https://www.cnblogs.com/glacierh/p/5689418.html)

6. [多阶Hash算法](https://www.owenzhang.net/blog/330.html)

7. [Hazard pointer](https://en.wikipedia.org/wiki/Hazard_pointer)

   

