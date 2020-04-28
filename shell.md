# 常用Shell脚本示例

### Sleep，定时echo

```shell
#! /bin/bash
# while loops
n=1

while (( $n <= 100 ))
do
    echo $n
    (( n++ ))
    sleep 2

done
```

### 生成火焰图（大概

```shell
#!/bin/bash
set -xeuo pipefail

PROCESS="VHRankingOnInsightTfsHub"

CMD=$1
case $CMD in
	-g) ft_get
	;;
	-f) flame_graph
	;;
	*) echo "Do nothing"
esac

function ft_get()
{
	ft get
	mv ft_local/*  ./
	sudo chmod 775 $PROCESS
}

function flame_graph()
{
	process_id=`ps -ef | grep $PROCESS | sed -n '1p' | awk '{print $2}'`
	echo "process="$PROCESS
	echo "process_id="$process_id

	perf_ret=`perf record -F 99 -a -g -p $process_id -- sleep 30`
	echo "perf_ret="$perf_ret

	# 生成折叠后的调用栈
	perf script -i perf.data &> perf.unfold
	# 生成火焰图
	./FlameGraph/stackcollapse-perf.pl perf.unfold &> perf.folded
	# 最后生成 svg 图
	./FlameGraph/flamegraph.pl perf.folded > $2
	echo $2" flamegraph created!"

	ft put $2
}
```

