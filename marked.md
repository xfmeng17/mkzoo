# 日常一些知识，遇到的BUG，听到的技术等记录

### 2021-02-28 HazardPointer初理解

目前知道hazard pointer试图解决的问题，但还是没有完全理解hazard pointer的实现细节，可能需要下一次继续理解了，这里先mark住相关的文章

- [无锁实现之Hazard Pointer (to be continue)](https://www.jianshu.com/p/b5cc79caa9a5)
- [Lock Free中的Hazard Pointer(上)](http://www.yebangyu.org/blog/2015/12/10/introduction-to-hazard-pointer/)
- [Hazard指针](https://blog.csdn.net/pongba/article/details/589864)
- [Lock-free 编程：A Case Study（下）](http://kaiyuan.me/2018/02/07/lock-free-prog2/)
- http://blog.kongfy.com/2017/02/hazard-pointer/
- 

### 2021-02-23 KFIFO再理解

> https://www.linuxidc.com/Linux/2016-12/137936.htm

![kfifo](https://github.com/xfmeng17/mkzoo/blob/master/picture/2021_02/kfifo.jpg?raw=true)

------

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

#### 2020-11-02 关于pb Utf8DebugString()打印\345\245\207引发的一些系列

1. 接入联调PB，发现Utf8DebugString() 打印bytes输出了这种 `\345\245\207\345\245\207\346\200\252\346\200\252\347\232\204`，十分疑惑

2. 首先不知都\345是啥，研究了下，`\`开头都是转义字符，`\ddd`是3位8进制，`\xhh`是2位16进制

   - [C\C++的转义字符](https://www.cnblogs.com/emanlee/archive/2010/05/14/1735274.html)

   - [C语言转义字符](http://c.biancheng.net/cpp/html/2890.html)

3. 那上面到底是啥中文？所以搞了个python程序：https://github.com/xfmeng17/tempwork/blob/master/python/decode8.py

4. 讲道理，`Utf8DebugString()`应该能识别啊？一看，pb类型是bytes，print的时候直接打印3位8进制，用`DebugString()`还是`Utf8DebugString()`没区别

5. 然后找了半天没找到直接把3位8进制UTF8转成中文的，自己想写个C++，发现，如果定义在代码内部，直接按转义字符，就是中文，如果重终端输入，就是**纯字符**，参见：https://github.com/xfmeng17/tempwork/blob/master/cpp/protobuf/utf8/utf8.cc

6. 还是需要仔细研究一下PB里面这个函数：

   1. https://sourcegraph.com/github.com/Samsung/TizenRT/-/blob/external/protobuf/src/google/protobuf/stubs/strutil.cc#L592
   2. https://www.fpstop.com/third/protobuf_string/

7. 03，继续读pb源码，发现`Utf8DebugString`和`DebugString`差别在于前者调用了

   - `TextFormat::Printer::SetUseUtf8StringEscaping(true)`
   - `class FastFaildValuePrinterUtf8Escaping`
   - `PrintString`
   - `Utf8SafeCEscape`
   - `CEscapeInternal` => 二级制到 \ddd 的形式，最大 字符串大小会扩 4倍

   但是，**目前存疑** 不应该是 `DebugString` 做二进制到 \ddd形式的转化，而`Utf8DebugString`直接打印二进制吗？？

------

#### [2020-10-27 什么是代码区、常量区、静态区（全局区）、堆区、栈区？](https://blog.csdn.net/u014470361/article/details/79297601)

1. 从低地址到高地址依次为：

   1. 代码区：存放代码，也即是CPU指令，只读

   2. 常量区：程序内部的常量，如 `int a = 100; string s = "hello"` 里面的100，hello，就是常量 

   3. 静态区（全局区）：静态变量和全局变量在一起，只初始化一次，程序全部结束后才释放

      - 静态变量，就是main或者其他函数内带static的变量

        - 全局变量，就是main外面的

   4. 堆区（向高地址扩展）

   5. 栈区（向低地址扩展）其实堆栈区中间是动态链接库区，用于在程序运行期间加载和卸载动态链接库

2. https://github.com/xfmeng17/tempwork/blob/master/c/test_char_arr.c 说明下为啥char * core char[] 不core？

   - char * 指向一个常量区，修改常量 segment fault
   - char[] str 定义了一个字符数组，在栈区，char[] str = "abcd"，是将常量区的"abcd"拷贝到str对应的地址空间中

3. C语言的static修饰符，令人迷惑：https://blog.csdn.net/qq_37858386/article/details/79064900

   	- 除一个例外，static就是作用域
   	- 这个例外就是，函数内部（main或者其他自定义函数）带static的，就不是作用域了（因为函数自带作用域，而是叫静态变量，静态变量就是带作用域的“全局变量”

4. 再复习下C程序的内存布局：http://c.biancheng.net/cpp/html/3180.html

------

#### [2020-06-10 引用在本质上是什么，它和指针到底有什么区别](http://c.biancheng.net/cpp/biancheng/view/3261.html)

- 引用只是对指针的封装，底层还是指针。对引用取地址，并不是这个引用的地址，而是引用所对应的变量的地址，但是并不是说引用自己不暂用空间，而是编辑器内部进行了转换，不让我们看到引用的地址。

- 那为什么会有引用这个东西呢，原因是设计语言的人觉得用指针写代码麻烦，有不能直接拷贝对象，就搞了个引用

- 这里也清晰了，既然底层是指针，那就只能指想在内存上的数据。对于临时变量（1，2，3）这种在寄存器上的，就不能引用。无法“声明”一个引用，因为他内部其实是一个指针！

- 那什么东东不放在内存上，或者更精确的说，是不能寻址呢？有2个：

  - 临时数据（基本类型的，如int），下面的是错的，但是func()返回结构体就是对的

    ```c++
    int n = 100, m = 200;
    int *p1 = &(m + n);    //m + n 的结果为 300
    int *p2 = &(n + 100);  //n + 100 的结果为 200
    bool *p4 = &(m < n);   //m < n 的结果为 false
    
    /********************/
    
    int func() {
        int n = 100;
        return n;
    }
    int *p = &(func());
    ```

  - 常量（或者是常量表达式）：常量和常量表达式会被认为是立即数，会被硬编码到指令中，虽然指令（即代码）也是加载到内存，但是是不能对他们进行内存寻址的

- 如果引用作为函数参数，那么调用函数传值时，如果传了一个常量或者常量表达式，就会遇到左值右值的报错了，如下：

  ```c++
  bool isOdd(int &n) {
      if(n/2 == 0){
          return false;
      }else{
          return true;
      }
  }
  int main() {
      int a = 100;
      isOdd(a);  //正确
      isOdd(a + 9);  //错误
      isOdd(27);  //错误
      isOdd(23 + 55);  //错误
      return 0;
  }
  ```

  编译tempwork/cpp/test下的ref.cpp，显示的错误：

  ```shell
  ref.cc: In function ‘int main()’:
  ref.cc:10:28: error: cannot bind non-const lvalue reference of type ‘double&’ to an rvalue of type ‘double’
    double v1 = volume(a, b, c);
  ```

  哈哈哈，左值右值，这不就清楚了嘛~~

- 右值引用 https://juejin.im/post/59c3932d6fb9a00a4b0c4f5b

------

### Q：为什么Rpckit框架下产生的core文件，有时候是进程名，有时候是一个rc_thw_xxx的名字？

- 曙光提的问题，看了下  `/proc/sys/kernel/core_pattern`显示为 `/data/coredump/core-%e-%p-%t`，即进程名，进程号，时间戳
- 框架内部使用 `pthread_setname_np` 设置了线程名，在子线程内产生core，如果设置了线程名（子线程默认继承主线程名字，也就是进程名，这里精确的表述应该是复写了自己的线程名）core文件是子线程的名字
- Rpckit里面搜索`rc_thw`能发现问题，`RcThreadHandlerRoutine`函数设置的。另一种方式是通过`prctl(PR_SET_NAME, thread_name)`来设置
- 测试代码：https://github.com/xfmeng17/tempwork/blob/master/cpp/test/threadcore.cc

### Q：git如何批量修改提交的user和email

- https://www.cnblogs.com/se7end/p/12704976.html

















