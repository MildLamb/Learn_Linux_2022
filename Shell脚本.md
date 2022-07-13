# Shell
## 体验一下shell
- 编写一个helloworld脚本，脚本格式以 #!/bin/bash 开头 （指定解析器）
```bash
[root@VM-4-16-centos scripts]# cat hello.sh 
#! /bin/bash
echo "hello,world"
```
- 执行   sh/bash 脚本文件位置  或者  直接写文件路径，但要求有执行权限x


# 变量
## 系统预定义变量
- 常用系统变量
  - $HOME
  - $PWD
  - $SHELL
  - USER
  - ...

## 自定义变量
- 基本语法
  - 定义变量：变量名=变量值， 注意，=后前后不能有空格（这种方式定义出来的都是字符串）
  - 提升作用域：export 变量名
  - 父shell全局变量可以影响子shell，子shell声明全局变量不会影响父shell
  - readonly定义只读变量， readonly 变量名=变量值
  - 撤销/删除变量  ， unset 变量名，只读变量不能使用 unset，关闭终端后自动从内存中删除

## 特殊变量
### $n   n：0-9
- 基本语法
  - $n ：n为数字，$0代表脚本名称，$1-9代表第一到九个参数，十以后的参数需要使用大括号包含，例如 ${10}

- 实际操作案例
```bash
# 编写脚本 paramater.sh
[root@VM-4-16-centos scripts]# cat parameter.sh 
echo '===========$n==========='
echo script name: $0
echo 1st paramater: $1
echo 2nd paramater: $2
echo 3th paramater: $3

# 执行脚本并添加参数
[root@VM-4-16-centos scripts]# . parameter.sh Kindred Gnar Neeko
===========$n===========
script name: -bash
1st paramater: Kindred
2nd paramater: Gnar
3th paramater: Neeko
```

### $#
- $# : 获取所有输入参数个数

### $*和$@
- $* : 代表命令行中所有的参数，$*会把所有参数看作一个整体
- $@ : 代表命令行中所有的参数，但$@更类似于数组，可以遍历其中的参数

### $?
- 类似于命令执行的返回值，正常返回0


# 运算符
- 基本语法
  - $((运算式)) 或 $[运算式]

- 案例：计算  (4+3) x 2 的值并复制给 sum
```bash
[root@VM-4-16-centos ~]# sum=$[(4+3)*2]
[root@VM-4-16-centos ~]# echo $sum
14
```

# 条件判断
- 基本语法
  - test condition
  - [ condition ]  （注意condition前后要有空格，中间的表达式也要空格）
  - 注意：条件非空即为true，[ kindred ]返回true，[]返回false

- 常用判断条件
  - 两个整数间比较
    - -eq 等于 （equal）
    - -ne 不等于 （not equal）
    - -lt 小于 （less than）
    - -le 小于等于 （less equal）
    - -gt 大于 （greater than）
    - -ge 大于等于 （greater equal）

  - 按照文件权限进行判断
    - -r 有读的权限 （read）
    - -w 有写的权限 （write）
    - -x 有执行的权限 （execute）

  - 按照文件的类型进行判断
    - -e 文件存在 （existence）
    - -f 文件存在并且是一个常规文件 （file）
    - -d 文件存在并且是一个目录 （directory）

- 多条件判断 （&&表示前一条命令执行成功时，才执行下一条命令，||表示上一条命令执行失败后，才执行下一条命令）
  - [ kindred ] && echo OK || echo notOK
```bash
[root@VM-4-16-centos scripts]# [ $a -gt 100 ] && echo "$a > 100" || echo "$a < 100"
50 < 100
```




```bash
[root@VM-4-16-centos scripts]# test $a = kindred
[root@VM-4-16-centos scripts]# [ $a = kindred ]
[root@VM-4-16-centos scripts]# echo $?
0
[root@VM-4-16-centos scripts]# [ -x test ]
[root@VM-4-16-centos scripts]# echo $?
1
[root@VM-4-16-centos scripts]# [ -x add.sh ]
[root@VM-4-16-centos scripts]# echo $?
0
[root@VM-4-16-centos scripts]# [ -e add.sh ]
[root@VM-4-16-centos scripts]# echo $?
0
[root@VM-4-16-centos scripts]# [ -f add.sh ]
[root@VM-4-16-centos scripts]# echo $?
0
[root@VM-4-16-centos scripts]# [ -d add.sh ]
[root@VM-4-16-centos scripts]# echo $?
1
```


# 实操案例，每天归档文件
- 归档脚本
```bash
#!/bin/bash

# 首先判断输入参数是否为1

if [ $# -ne 1 ]
then
	echo "参数个数错误!应该输入一个参数，作为归档目录"
	exit
fi

# 从参数中获取目录名称
if [ -d $1 ]
then
	echo
else
	echo
	echo "目录不存在"
	echo
	exit
fi

DIR_NAME=$(basename $1)
DIR_PATH=$(cd $(dirname $1);pwd)

# 准备创建归档文件名称，获取日期
DATE=$(date +%y%m%d)

# 创建名称
FILE=archive_${DIR_NAME}_${DATE}.tar.gz
DEST=/usr/archive/${FILE}

# 开始归档目录文件

echo "准备归档..."
echo

tar -czf $DEST $DIR_PATH/$DIR_NAME

if [ $? -eq 0 ]
then
	echo
	echo "归档完成"
	echo "归档文件为:$DEST"
else
	echo "归档出现问题"
	echo 
fi

exit
```
- 设置定时任务
```bash
[root@VM-4-16-centos scripts]# crontab -e
0 2 * * * /usr/scripts/daily_archive.sh /usr/scripts
```


# 实操案例，发送消息
利用Linux自带的mesg和write工具，向其他用户发送消息  
需求：实现一个向某用户快速发送消息的脚本，输入用户名作为第一个参数，后面直接  
跟要发送的消息。脚本需要检测用户是否登录在系统中，是否开启消息功能，以及当前  
消息是否为空

## 代码实现
```bash
#!/bin/bash

# 查看用户是否登录
login_user=$(who | grep -i -m 1 $1 | awk '{print $1}')

if [ -z $login_user ]
then 
	echo "$1 不在线"
	echo "脚本退出"
	exit
fi


# 查看用户是否开启消息功能
is_allow=$(who -T | grep -i -m 1 $1 | awk '{print $2}')

if [ $is_allow != "+" ]
then
	echo "$1 未开启消息功能"
	echo "脚本退出"
	exit
fi

# 查看消息是否为空
if [ -z $2 ]
then
	echo "没有消息发送"
	echo "脚本退出"
	exit
fi


# 发送消息
# 从参数中获取消息
whole_msg=$(echo $* | cut -d " " -f 2-)

# 获取用户登录的终端
user_terminal=$(who | grep -i -m 1 $1 | awk '{print $2}')

# 写入要发送的消息
echo $whole_msg | write $login_user $user_terminal

if [ $? != 0 ]
then
	echo "发送失败!"
else
	echo "发送成功"
fi
exit
```
```bash
[root@VM-4-16-centos scripts]# ./send_msg.sh mildlamb Mildlamb and Wildwolf
发送成功
```

