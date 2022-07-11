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

```
