

# 常用Shell脚本示例

### #*/和%/

> https://blog.csdn.net/weixin_33699914/article/details/92900047?utm_medium=distribute.pc_relevant.none-task-blog-title-2&spm=1001.2101.3001.4242

${string#substring}Strip shortest match of $substring from front of $string

${string%substring}Strip shortest match of $substring from back of $string

#*/ 代表删除从前往后最小匹配的内容，\*统配字符，XXXX/

%/* 代表删除从后往前最小匹配的内容, /XXXXX

### while

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

### case

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

### rename

v 显示文件重命名的细节

n 不执行重命名，但会模拟执行重命名，并显示会出现的情况，例如是否会有同名文件冲突等。在重命名前测试很有用。

f 强制覆盖同名文件

```shell
touch a.txt
touch b.txt
touch c.txt
rename -v 's/.txt/.log/' *.txt
```





