# Nginx 使用

## Nginx安装

nginx运行命令

```Bash
# 查看nginx是否运行
ps -ef | grep nginx

# 重新载入nginx配置文件
nginx -s reload

# 停止nginx 快速关闭
nginx -s stop

# 重启nginx
nginx -s reopen

# 优雅的关闭 nginx
nginx -s quit

# 帮助
nginx -? | -h

# 使用备用配置文件而不是默认文件
nginx -c <file>

# 测试配置文件：nginx检查配置是否有正确的语法，然后尝试打开配置中引用的文件
nginx -t
```

## Nginx配置文件

> 参考资料：
> * [CSDN Java高知 - Nginx配置文件（nginx.conf）配置详解](https://blog.csdn.net/tjcyjd/article/details/50695922)
> * [简书 忘净空 - nginx配置详解](https://www.jianshu.com/p/1af680730850)

__说明：以下内容为上面两篇博客内容，自己记录为笔记以便学习__ （如要转载请自行联系作者）

nginx 配置文件结构

```Bash

main
events   {
  ....
}
http        {
  ....
  upstream myproject {
    .....
  }
  server  {
    ....
    location {
        ....
    }
  }
  server  {
    ....
    location {
        ....
    }
  }
  ....
}

```

从文件结构可知配置文件主要由一下六部分组成：

* 1、main(全局设置)
* 2、events(nginx工作模式)
* 3、http(http设置)
* 4、sever(主机设置)
* 5、location(URL匹配)
* 6、upstream(负载均衡服务器设置)
