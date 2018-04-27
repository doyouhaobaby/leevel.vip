# PHP 处理浮点数高精度计算函数 Binary Calculator 用于金额计算

## 支持函数

PHP 为任意精度数学计算提供了二进制计算器（Binary Calculator），它支持任意大小和精度的数字，以字符串形式描述。

[PHP 官方文档](http://php.net/manual/en/ref.bc.php)

```
bcadd  加法  addition [əˈdɪʃən]
bccomp 比较 compare [kəmˈper] 
bcdiv 相除 division  [dɪˈvɪʒən] 
bcmod  求余数 mod [mɑ:d]
bcmul 乘法 multiplication [ˌmʌltəplɪˈkeʃən]
bcpow 次方 power [ˈpaʊɚ]
bcpowmod 先次方然后求余数 power mod
bcscale 给所有函数设置小数位精度 scale [skel]
bcsqrt 求平方根 square root [skwɛr rut] 
bcsub 减法 subtraction [səbˈtrækʃən]
```

## 使用例子

下面是一些基本使用的例子，例子摘自 PHP 官方文档。

```php
<?php
$a = '1.234';
$b = '5';

echo bcadd($a, $b);     // 6
echo bcadd($a, $b, 4);  // 6.2340

// default scale : 3
bcscale(3);
echo bcdiv('105', '6.55957'); // 16.007

// this is the same without bcscale()
echo bcdiv('105', '6.55957', 3); // 16.007

?>
```
