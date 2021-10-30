

[TOC]



#### shell脚本

作用：linux内核再硬件服务器上，shell脚本可以操作linux内核



##### 1.helloworld

```shell

touch helloworld.sh
vi helloworld.sh

#!/bin/bash
echo "helloworld my xujiaqi"

chmod 777 helloworld.sh
./helloworld.sh

```



##### 2.多命令处理

```shell

touch banzhang.sh
vi banzhang.sh

#!/bin/bash
cd /opt/mytest/
touch banzhang.txt
echo "i love you" >> banzhang.txt

结果：生成了"banzhang.txt"文件，并且再该文件中输入了字符串"i love you"

```



##### 3.常用的系统变量

```shell

[root@VM-0-17-centos mytest]# echo $HOME
/root
[root@VM-0-17-centos mytest]# echo $PWD
/opt/mytest
[root@VM-0-17-centos mytest]# echo $SHELL
/bin/bash
[root@VM-0-17-centos mytest]# echo $USER
root

```



##### 4.自定义变量

```shell

[linux@localhost datas]$ A=1
[linux@localhost datas]$ echo $A
1
[linux@localhost datas]$ unset A    ## 撤销对变量A的定义

定义全局变量：export 变量名

```



##### 5.特殊变量

###### 5.1 $n：给脚本传参数

```shell

touch test.sh
vi test.sh

#!/bin/bash
echo "$0 $1 $2 $3"

bash test.sh one tow   ## 输出结果为：test.sh one tow

注：$0代表脚本名称 之后的代表传入的参数

```



###### 5.2 $#：获取输入参数的个数，用于循环

```shell

#!/bin/bash
echo "$0 $1 $2 $3"
echo $#

```



###### 5.3 $* 和$@：获取所有参数，把所有参数当成一个参数

```shell

[linux@localhost datas]$ bash parameter.sh test1 test2
parameter.sh test1 test2 
2
test1 test2
test1 test2

```



###### 5.4 $?：最后一次执行命令的状态，0：正确执行 



##### 6.运算符

- 1.$((运算式)) 或 $[运算式]
- 2.expr +,-,\*,/,% 加减乘除取余   注：*expr运算符间要有空格*

````shell

# 计算2+3
[linux@localhost datas]$ expr 2 + 3
5

# 计算（2+3）*4
## 方式1
[linux@localhost datas]$ expr `expr 2 + 3` \* 4
20

## 方式2
[linux@localhost datas]$ s=$[(2+3)*4]
[linux@localhost datas]$ echo $s
20

````



##### 7.条件判断

 具体使用：[condition] (注意 condition前后要有空格) 

多条件判断： && 、 || 

![](https://gitee.com/domineering_red_tide/image/raw/master/image/20210629205616.png)



```shell

# 判断23是否大于2
[linux@localhost datas]$ [ 23 -gt 2 ]
[linux@localhost datas]$ echo $?
0

# 判断helloworld.sh是否有写入权限
[linux@localhost datas]$ [ -w hellowrld.sh ]
[linux@localhost datas]$ echo $?
1

# 判断目录中文件是否存在
[linux@localhost datas]$ [ -e /home/linux/datas ]
[linux@localhost datas]$ echo $?
0

```



##### 8.流程控制

###### 8.1 if语句

```shell

#!/bin/bash
if [ $1 -eq 1 ]
then
	echo "班长真帅"
elif [ $1 -eq 2 ]
then
	echo "班长真丑"
fi
[linux@localhost datas]$ bash if.sh 2
班长真丑

```



###### 8.2 case语句

```shell

#!/bin/bash
case $1 in
1)
	echo "班长"
;;
2)
	echo "学习委员"
;;
3)
	echo "体育委员"
;;
esac
[linux@localhost datas]$ bash case.sh 2
学习委员

```



###### 8.3 for循环

```shell

#!/bin/bash
s=0
for((i=1;i<=100;i++))
do
	s=$[$s+$i]	
done
echo $s
[linux@localhost datas]$ bash for.sh
5050

```



```shell

#!/bin/bash
for i in $*
do
	echo $i
done
[linux@localhost datas]$ bash for2.sh 1 2
1
2

```



###### 8.4 while循环

```shell

#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
	s=$[$s + $i]
	i=$[$i + 1]
done
echo $s

[linux@localhost datas]$ bash while.sh
5050

```



##### 9.函数

###### 9.1 常用函数read：读取控制台输入

注：read函数 -p 指定读取值时的提示符  -t 指定读取值时等待的时间（秒）

```shell

# 提示7秒内，读取控制台输入的名称
#!/bin/bash
read -t 7 -p "在7s内请输入你的名字" NAME
echo $NAME

[linux@localhost datas]$ bash read.sh
在7s

```



###### 9.2 系统函数

-  basename：basename [string / pathname] [suffix] （描述：basename命令会删掉所有的前缀包括最后一个‘/’字符，然后将字符串显示出来）

```shell

# 方式1
[linux@localhost datas]$ basename /home/linux/banzhang.txt
banzhang.txt

# 方式2
[linux@localhost datas]$ basename /home/linux/banzhang.txt .txt
banzhang

```



- dirname：dirname 文件绝对路径 （描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））

```

[linux@localhost datas]$ dirname /home/linux/banzhang.txt
/home/linux

```



###### 9.3 自定义函数

```shell

# 格式
[ function ] funname[()]
{
	Action:
	[return int;]
}
funname

```



```shell

# DESC 计算输入两个参数的值
#!/bin/bash
function sum(){
	s=0;
	s=$[$1 + $2]
	echo $s
}
read -p "input your param1:" P1
read -p "input your param2:" P2
sum $P1 $P2

[linux@localhost datas]$ bash sum.sh
input your param1:1
input your param2:2
3

```



##### 10.shell工具

###### 10.1 cut切割字符串

cut命令从文件的每一行剪切字节，字符和字段并将这些字节，字符和字段输出
cut [选项参数] filename

- -f 列号，提取第几列
- -d 分隔符，按照指定分隔符分隔列

```shell

# DESC 切割cut.txt第一列
touch cut.txt
vi cut.txt
dong shen
guan zhen
wo wo
lai lai
le le

[linux@localhost datas]$ cut -d " " -f 1 cut.txt
dong
guan
wo
lai
le

# DESC 获取包含guan行的第一个单词
# cat是查看的意思。获取到文件内容
[linux@localhost datas]$ cat cut.txt | grep guan | cut -d " " -f 1
guan

```



###### 10.2 sed编辑文件器

sed是一种流编辑器，它一次处理一行内容，处理时，把当前处理的行存储在临时缓冲区中，成为“模式空间”,接着sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕，接着处理下一行，这样不断重复，知道文件末尾，文件内容并没有改变，除非你使用重定向存储输出

sed [选项参数] ‘command’ filename

- -e 直接在指令列模式上进行sed的动作编辑

命令功能描述

- a 新增
- d 删除
- s 查找并替换

```shell

touch cut.txt
vi cut.txt
dong shen
guan zhen
wo wo
lai lai
le le

# DESC 在第二行后增加mei nv字符
# 2a表示第2行增加的意思
[linux@localhost datas]$ sed -e "2a mei nv" sed.txt
dong zhen
guan zhen
mei nv
wo wo
lai lai

```



###### 10.3 awk类似前面的cut用法

awk 一个强大文件分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理 

awk [选项参数] 'pattern1 {action1} pattern2{action2}..' filename

- -F 指定输入文件分隔符
- -v 赋值一个用户定义变量



###### 10.4 sort排序

sourt [选项] (参数)

