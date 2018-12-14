# Mac电脑使用小技巧

查看端口占用

```Bash
# 查看占用的端口
lsof -i:8080

# 杀掉占用端口的进程
kill -9 PID
```

Mac启用自带FTP服务

> [Mac启用自带FTP服务](https://blog.csdn.net/leifjacky/article/details/52189895)

```Bash
# 开启 FTP Server
sudo -s launchctl load -w /System/Library/LaunchDaemons/ftp.plist

# 关闭 FTP Server
sudo -s launchctl unload -w /System/Library/LaunchDaemons/ftp.plist
```