# Linux 文件基本属性

Linux系统是一种典型的多用户系统，不同的用户处于不同的地位，拥有不同的权限。

## 查看文件权限

使用 `ls -l` 或者 `ll` 命令来查看文件权限。

```
/data/codes/linux » ll
total 0
drwxr-xr-x  3 dyhb  staff    96B  5  3 11:44 cat
-rw-r--r--  1 dyhb  staff     0B  5  4 18:31 test.md
```

在Linux中第一个字符代表这个文件是目录、文件或链接文件等等。
* 当为[ d ]则是目录
* 当为[ - ]则是文件；
* 若是[ l ]则表示为链接文档(link file)；
* 若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
* 若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

接下来的字符中，以三个为一组，且均为『rwx』 的三个参数的组合。其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。

分析一下上面的 `test.md` 文件,依次分析：

* `-`  表示是一个文件
* `r`  表示拥有属主可读权限，值为 4
* `w` 表示拥有属主可写权限，值为 2
* `-` 表示没有属主执行权限 0，值为 0
* `r`  表示拥有属组可读权限，值为 4
* `-` 表示没有属组可写权限，值为 0
* `-` 表示没有属组执行权限 0，值为 0
* `r`  表示拥有其他用户可读权限，值为 4
* `-` 表示没有其他用户可写权限，值为 0
* `-` 表示没有其他用户执行权限 0，值为 0

最终得到其权限只为 `644`,3 个一组分别相加，`-` 表示没有权限的意思其值为 0。

## Linux 文件属主和属组

对于文件来说，它都有一个特定的所有者，也就是对该文件具有所有权的用户。同时，在Linux系统中，用户是按组分类的，一个用户属于一个或多个组。文件所有者以外的用户又可以分为文件所有者的同组用户和其他用户。

上面的例子中，`test.md` 的文件：

```
dyhb = 属主
staff = 属组
```

## 更改文件属性

### chgrp 更改文件属组

```
chgrp [-R] 属组名 文件名
```

### chown：更改文件属主，也可以同时更改文件属组

```
chown [–R] 属主名 文件名
chown [-R] 属主名：属组名 文件名
```

我们修改 `test.md` 的属主 为 root 和属组为 staff,这里没有修改.

```
sudo chown root:staff test.md
/data/codes/linux » ll 
total 0
drwxr-xr-x  3 dyhb  staff    96B  5  3 11:44 cat
-rw-r--r--  1 root  staff     0B  5  4 18:31 test.md
```

### chmod：更改文件9个属性

Linux文件属性有两种设置方法，一种是数字，一种是符号。

Linux文件的基本权限就有九个，分别是 owner/group/others 三种身份各有自己的 read/write/execute 权限。

数字来代表各个权限，各权限的分数对照表如下：
* r:4
* w:2
* x:1

每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如 `test.md`： [-rw-r--r--] 分数则是：

```
owner = rw- = 4+2+0 = 6
group = r-- = 4+0+0 = 4
others = r-- = 4+0+0 = 0
```

我们这里修改 `test.md` 修改其权限为 777。

```
sudo chmod 777 test.md
/data/codes/linux » ll
total 0
drwxr-xr-x  3 dyhb  staff    96B  5  3 11:44 cat
-rwxrwxrwx  1 root  staff     0B  5  4 18:
```

### 符号类型改变文件权限

通过上面的学习基本上就九个权限分别是(1)user (2)group (3)others三种身份啦！ 那么我们就可以藉由u, g, o来代表三种身份的权限！

```
sudo chmod u=rwx,g=rx,o=r test.md
/data/codes/linux » ll
total 0
drwxr-xr-x  3 dyhb  staff    96B  5  3 11:44 cat
-rwxr-xr--  1 root  staff     0B  5  4 18:31 test.md
```

### 删除文件权限

删除所有执行权限，代码如下：

```
sudo chmod a-x test.md
/data/codes/linux » ll
total 0
drwxr-xr-x  3 dyhb  staff    96B  5  3 11:44 cat
-rw-r--r--  1 root  staff     0B  5  4 18:31 test.md
```

## 后记

学习记录一下，又有了新的认识了，非常给力。

## 参考文章

* [Linux 文件基本属性初认识](https://www.linuxidc.com/Linux/2018-03/151466.htm)
* [Linux系统中文件的基本属性](http://blog.51cto.com/vinsent/1951574)