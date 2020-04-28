

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

### Case

```shell
#!/bin/bash
set -xeuo pipefail

CMD=$1
case $CMD in
	-g) echo "-g"
	;;
	-f) flame_graph
	;;
	*) echo "Do nothing"
esac

function flame_graph()
{
	echo "flame_graph"
}
```

### data format

```shell
MESSAGE=`date +"%Y-%m-%d %T"`
```

