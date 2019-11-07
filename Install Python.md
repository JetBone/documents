# 搭建Python环境整理

在服务器上搭建python环境的时候，由于服务器端安装的是最小的linux(就是要啥没啥，我也不知道需要啥，我也不知道有些啥，总结，我啥也不知道)，所以很多组件都没有安装好，直接安装python会出问题，这里简单进行一些记录

## 编译python源码前的一些准备工作

``` bash
# OS: Centos 7
# 这些安装都是

# 安装gcc，用于编译源代码
yum -y install gcc

# 安装zlib，用于编译python(也许其它东西也会用到它)
# 编译python的时候会报zlib没有的错误，所以需要安装
yum -y install zlib-devel

# 安装ssl，否则编译出来的python将无法使用pip安装第三方包
yum -y install openssl-devel

# 安装bzip2，否则编译出来的python在调用的时候会报找不到_bz2 module
yum -y install bzip2-devel
```

## 完成准备工作后使用源码安装python

``` bash
# 随便找个目录解压python源码包，这里填的是我的python源码路径，会解压到当前执行命令的目录下，会出现一个名字为Python-3.6.4的目录，里面是解压好的源码
tar -zxvf ../../soft/Python-3.6.4.tgz

# 进入解压出来的源码目录内
cd Python-3.6.4/

# 执行./configure，我这里加上了prefix参数，用于指定python最终的安装目录，不指定安装的是默认目录，除非有特殊运维要求，否则也没必要指定，linux目录都分的很好，我这里的指定目录是我随便写的，我并不想安装到home下蛤蛤
./configure --prefix=/home/python

# 执行make
make

# 执行make install
make install
```

## 配置环境变量

由于我们是使用源码安装的python，完成的工作只是将python安装到了我们需要的目录(如果之前指定了)，但是系统的环境变量都是不会给你配好的(如果使用的是rpm包安装的，其实会直接帮你建立好软连接，不需要配置环境变量，但是源码安装不行)，所以最后我们还需要手动配置一下环境变量

具体配置到哪个文件里，实际上因人而已，或者有特殊的运维需求
这里简单介绍一下

  1. /etc/profile 如果配置到这个文件里，则代表的是系统变量，所有人都能访问到这里的东西
  2. /home/user/.bash_profile 如果配置到这个文件里，则代表的是用户的系统变量，只有这个目录下的用户才能访问到

``` bash
export PYTHON_HOME=balabala
export PATH=${PYTHON_HOME}/bin:${PYTHON_HOME}/bin
```
