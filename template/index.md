# 模板

QueryPHP 内置三种方式的模板引擎，一种是类似于 `smart` 模板的 Code 语法，一种是 HTML 标签式的 Node 语法，例外还有一种类 Javascript 的语法与 Twig 比较相似。

 * code 语法，侧重简单实用
 * Node 语法，严谨务实
 * JS 语法，现代潮流

例外，三种语法随心嵌套，随意使用，QueryPHP 模板引擎底层分析器和编译器非常健壮，完美运行 8 年。

QueryPHP 模板引擎技术来自于 Jecat,一款无与伦比的技术大餐，有幸在 2010 接触到这个框架，通过这个框架学到了很多。

## Code 语法

``` html
{$name}

{if $name == 'You'}
    欢迎进入 QueryPHP 开发者世界！
{/if}
```

## Node 语法

``` html
<if condition="$name eq 'You'">
    欢迎进入 QueryPHP 开发者世界！
</if>
```

## 现代化类 JS 语法

``` html
{{i + 1}}
```

## 拒绝交叉

下面这种写法就是错误的，模板引擎将无法正确解析。

``` html
<$name>

{if condition="$name eq 'You'"}
    欢迎进入 QueryPHP 开发者世界！
{/if}
```

## PHP 方式

如果你不习惯使用使用内置的模板引擎，你也可以完全使用 PHP 自生来写。

``` php
<?php if ($name == 'You'): ?>
    欢迎进入 QueryPHP 开发者世界！
<?php endif; ?>
```

## 扩展支持

为了更好地追随时代的潮流，系统扩展支持了流行的模板引擎 Twig，例外还支持基于 v8js 的服务端运行 Javascript 代码，以及 PHPUI 桌面应用开发。

第三方的教程留给大家自行探索，其乐无穷。
