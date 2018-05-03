# Nginx日志分析命令

公司架构师在分析 Nginx 日志做接口响应请求统计，这里我不懂有些疑惑查阅了相关资料整理一些以后方便学习的资料，方便以后做一些性能相关的处理。

## Nginx 日志格式

打开 Nginx 的配置文件，可以查看日志的格式。

```
vim /server/nginx-1.6.2/nginx.conf
/log_format
```

可以看到我们的默认日志格式为，日志格式可以对照着下面得到的日志一一对应。比如，`$upstream_response_time` 表示服务器响应时间，可以用来做性能优化的关键参数。

```
 log_format  main  '$remote_addr - $remote_user [$time_local] $request '
      ' | $status | $body_bytes_sent | $http_referer |'
      ' | $uri |'
      ' | $http_user_agent | $http_x_forwarded_for |'
      ' $request_body |'
      ' $request_uri |'
      ' $request_time |  $upstream_response_time';
```

**$request_time** 说明：

指的就是从接受用户请求的第一个字节到发送完响应数据的时间，即包括接收请求数据时间、程序响应时间、输出响应数据时间。

**$upstream_response_time** 说明：

是指从Nginx向后端（php-cgi)建立连接开始到接受完数据然后关闭连接为止的时间。$request_time 比$upstream_response_time值大，所有我们需要在 log_format 中加入 $upstream_response_time 用来后期做性能优化分析。

## 日志分析

打开日志查看

```
cd /var/log/nginx
```

打开前看看日志总记录数

```
cat pc-dhb168.access.log |wc -l
44968
```

或者

```
wc -l pc-dhb168.access.log
44968 pc-dhb168.access.log
```

得到的日志每行一条：

```
cat pc-dhb168.access.log
...
10.0.2.2 - - [03/May/2018:10:41:05 +0800] GET /?module=Manager&controller=Costprice&action=lists&type=json&page=0&pageSize=30 HTTP/1.1  | 200 | 5136 | http://admin.new-dhb.com/Manager/Home/index | | /index.php | | Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36 | - | - | /?module=Manager&controller=Costprice&action=lists&type=json&page=0&pageSize=30 | 0.089 |  0.089
10.0.2.2 - - [03/May/2018:10:41:07 +0800] GET /?module=Manager&controller=Warehousing&action=warehousing&_m_=1&_js_init_= HTTP/1.1  | 200 | 92741 | http://admin.new-dhb.com/Manager/Home/index | | /index.php | | Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36 | - | - | /?module=Manager&controller=Warehousing&action=warehousing&_m_=1&_js_init_= | 0.307 |  0.306
10.0.2.2 - - [03/May/2018:10:41:08 +0800] GET /?module=Manager&controller=Ships&action=index&_m_=1&_js_init_= HTTP/1.1  | 200 | 85054 | http://admin.new-dhb.com/Manager/Home/index | | /index.php | | Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36 | - | - | /?module=Manager&controller=Ships&action=index&_m_=1&_js_init_= | 0.171 |  0.171`
... and more
```

## 统计接口调用次数

比如我要统计 `GET /?module=Manager&controller=BaseCoupon&action=index` 接口在某天的调用次数，则可以使用如下命令

```
cat pc-dhb168.access.log |grep 'GET /?module=Manager&controller=BaseCoupon&action=index' |wc -l
7
```
?> 其中 cat 用来读取日志内容，grep 进行匹配的文本搜索，wc 则进行最终的统计。

可以将统计结果写入文件，追加一个 `> hello.txt`,后面也是一样的。

```
cat pc-dhb168.access.log |grep 'GET /?module=Manager&controller=BaseCoupon&action=index' |wc -l > hello.txt
```

## 统计接口调用次数排行

这里 awk 是按照空格把每一行日志拆分成若干项,索引从 `$1` 开始，其中 `$7` 对应的就是URL，当然具体对应的内容和使用nginx 时设置的日志格式有关。

```
10.0.2.2 - - [03/May/2018:10:41:08 +0800] GET /?module=Manager&controller=Ships&action=index&_m_=1&_js_init_= HTTP/1.1  | 200 | 85054 | http://admin.new-dhb.com/Manager/Home/index | | /index.php | | Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36 | - | - | /?module=Manager&controller=Ships&action=index&_m_=1&_js_init_= | 0.171 |  0.171`
```

基于空格分析如下：

```
$1 10.0.2.2
$2  -
$3  -
$4  [03/May/2018:10:41:08
$5  +0800]
$6  GET
$7  /?module=Manager&controller=Ships&action=index&_m_=1&_js_init_=
... and more
```

基于上面的分析，我们对 `$7` 请求 URL 进行统计：

```
awk '{a[$7]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n30"}' pc-dhb168.access.log
989 /Manager/Home/index
848 /?module=Manager&controller=Home&action=getMsgInternal
702 /Auth/Index/manager_index
619 /
568 /static/images/loading_new.gif
519 /static/images/loadmenu.gif
512 /static/images/guide-13.jpg
512 /static/images/guide-12.jpg
512 /static/images/guide-11.jpg
511 /static/images/guide-16.jpg
511 /static/images/guide-15.jpg
511 /static/images/guide-14.jpg
511 /static/images/guide-10.jpg
511 /static/images/guide-09.jpg
511 /static/images/guide-08.jpg
511 /static/images/guide-07.jpg
511 /static/images/guide-06.jpg
511 /static/images/guide-05.jpg
511 /static/images/guide-04.jpg
511 /static/images/guide-03.jpg
511 /static/images/guide-02.jpg
511 /static/images/guide-01.jpg
499 /static/images/guide-icon.png
456 /favicon.ico
173 /Manager/Accounts/logout
170 /Auth/Index/handleManagerLogin
... and more
``` 

又比如统计访问最多 IP:

```
awk '{a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n30"}' pc-dhb168.access.log
44964 10.0.2.2
4 127.0.0.1
```

?> sort 用来排序，`-k1` 表示要把进行排序的第一列作为数字看待，并且结果倒序排列,`-nr` 表示已经读取的记录数，`head -n30` 意为取排名前 30 的结果。 

## 统计响应时间大于某个时间的请求

$NF 表示的最后一个Field（列），即输出最后一个字段的内容。

```
cat pc-dhb168.access.log |awk '($NF>1){print $6 $7 $8 " = " $NF}' > result.txt
```

统计请求时间大于 1 秒的数量

```
cat pc-dhb168.access.log |awk '($NF>1){print $6 $7 $8 " = " $NF}' |wc -l
724
```

统计总数量

```
cat pc-dhb168.access.log  |wc -l
44968
```

统计大于 1 秒请求占用比例

```
echo "scale=6;724/44968"|bc
.016100
```

?> scale=2 设小数位，2 代表保留两位, bc 是任意精度计算器语言。

## grep 打印匹配的前后几行 

有时候我们需要查找某个特定请求的前后几行的请求，以观察用户的关联操作情况。

```
grep -C 5 'controller=Costprice' pc-dhb168.access.log //打印匹配行的前后5行
grep -A 5 'controller=Costprice' pc-dhb168.access.log //打印匹配行的后5行
grep -B 5 'controller=Costprice' pc-dhb168.access.log //打印匹配行的前5行
```

## 结束语

文章先写到这里，后面遇到问题，慢慢添加更多，必须要亲自实践才能够学到知识。

## 参考文章
* [常用linux命令总结（二）nginx日志分析命令](https://blog.csdn.net/song19890528/article/details/48007917)
* [awk分析nginx日志里面的接口响应时间](http://jxausea.iteye.com/blog/2148397)
* [nginx优化之request_time 和upstream_response_time差别](http://blog.sina.com.cn/s/blog_4f9fc6e10102uxib.html)
* [Linux Awk使用案例总结（nginx日志统计，文件对比合并等）](https://www.cnblogs.com/276815076/p/6410179.html)
* [Linux bc 命令](http://www.runoob.com/linux/linux-comm-bc.html)