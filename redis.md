## Redis相关知识

### Hyperloglog

- 做基数计数（cardinality counting）的，就是给定一个集合，提供两个操作：

  - 返回集合中元素的个数
  - 给俩集合，返回merge后的元素个数

- 技术发展是Map，B树，bitmap（准确的），Linear Counting，LogLog Counting，HyperLogLog Counting（不准确的，基于概率的，但是能记录的多）

- 不准确的算法都是哈希+概率，一个比一个NB，精妙。但是算法/数学都不深究，大概了解是啥，用到了再学就赶趟。

- Redis自带HyperLogLog Counting，操作如下：

  ```shell
  127.0.0.1:6379> pfadd m1 1 2 3 4 1 2 3 2 2 2 2
  (integer) 1
  127.0.0.1:6379> pfcount m1
  (integer) 4
  127.0.0.1:6379> pfadd m2 3 3 3 4 4 4 5 5 5 6 6 6 1
  (integer) 1
  127.0.0.1:6379> pfcount m2
  (integer) 5
  127.0.0.1:6379> pfmerge mergeDes m1 m2
  OK
  127.0.0.1:6379> pfcount mergeDes
  (integer) 6
  ```

- 这里主要区分下和BloomFilter布隆过滤器的应用区别

  - HyperLogLog Counting 是集合中元素个数的统计，适用于统计UV等
  - BloomFilter是元素是否在集合中，适合做曝光过滤