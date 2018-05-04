# Linux 开机关机重启命令

一般来说 Linux 服务器是不需要关机的，这一点平时用 mac 就体会出来了。

## 关机命令

正确的关机流程为：sync > shutdown > reboot > halt，shutdown 关机指令，你可以 man shutdown 来看一下帮助文档。

```
sync  //将数据由内存同步到硬盘中。
Shutdown –h now // 立马关机
Shutdown –h 20:25 // 系统会在今天20:25关机
reboot //重启
halt // 关闭系统
```

## 参考文章

* [Linux 系统启动过程](http://www.runoob.com/linux/linux-system-boot.html)