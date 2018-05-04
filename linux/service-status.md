# Linux 查看服务与进程状态

这里记录记录 Linux 服务和进程一些技术文档，以便于日后查询使用。

## 服务 

服务相当于主机能够对外提供的功能，必须要有相应的守候进程来监听。

## 服务与进程 

服务是多个进程的集合，服务可能包含多个进程。

* 进程是一个独立的可调度的活动；
* 进程是一个抽象实体，当它执行某个任务时，要分配和释放各种资源；
* 进程是可以并行执行的计算单位；
* 进程是一个具有一定独立功能的程序关于某个数据集合的一次运行活动；

## PS 命令查询进程

PS 支持的参数非常多，常用参数如下，帮组命令 `man ps`：

*  -A 列出所有的进程（等价于-e） 
* -w 显示加宽可以显示较多的资讯 
*  -au 显示较详细的资讯 
* -aux 显示所有包含其他使用者的行程

常用的通过进程查看服务的命令如下：

```
ps -aux ｜ grep php-fpm
```
## Service 命令查看服务

### 查看所有服务运行状态

```
service –status-all 
```

### 查看服务状态

```
service mysql status
```

### 启动重启和关闭

```
service mysql start
service mysql restart
service mysql stop
```

## 参考文章
* [小何讲进程： Linux进程的基本概念](https://blog.csdn.net/rl529014/article/details/51280018)
* [linux的守护进程与服务-概念](https://blog.csdn.net/breeze_life/article/details/9409243)