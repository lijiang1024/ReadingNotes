# CentOS 常用命令

## 查看CentOS版本

```Bash
# rpm -q centos-release
centos-release-6-8.el6.centos.12.3.x86_64
```

## ssh连接远程Linux服务器

```Bash
# 连接
# ssh <username>@<ip>
ssh admin@58.48.109.84

# 断开连接
logout

# 或者

quit
```

## 修改完linux bashrc文件之后，如何不重启系统使其生效

```Bash
source .bashrc
```

## 查看是否安装某个工具

```Bash
# command -v <Tool Command>
command -v unzip
```

## 查找Nginx安装的目录

```Bash
whereis nginx
```

## 查找服务是否启动

```Bash
ps -ef | grep nginx
```

## telnet 测试端口号

```Bash
# telnet IP 端口 或者 telnet 域名 端口

telnet 58.48.109.84 3306
```

## 查看CPU资源

> [Centos 下查看服务器CPU的信息](https://blog.csdn.net/ghj1976/article/details/6158953)

```Bash
# 查看有几个逻辑CPU，以及CPU型号
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

# 实际是有几个CPU
cat /proc/cpuinfo | grep physical | uniq -c

# 2  Intel(R) Xeon(R) CPU E7- 4870  @ 2.40GHz
```