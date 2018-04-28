# Laravel 中间件实现方式之管道模式 Pipeline 解析及设计优化

## Laravel 中间件

Laravel 框架的中间件主要实现核心为 PHP 内置的 `array_reduce` 函数，利用闭包先定义执行代码的特性，生成了一个嵌套闭包，通过 `$next` 实质是下一个执行的 pipe 闭包，其表现形式与迭代器几乎一致 next 移动到下一个元素。

我这里将这个代码抽离一份代码可以执行代码，大家可以看看，结合后面的参考理解。

```php
<?php
// Xiangmin Liu
$end = function($passable) {
    echo 'The end' . PHP_EOL;
    $passable += 6; // +6
    return $passable;
};

$firstSlice = function ($passable) use ($end) {
    return call_user_func($end, $passable);
};

$pipes = [
    function(Closure $next, $passable) {
        echo 'First' . PHP_EOL;
        $passable ++; // +1
        return $next($passable);
    },

    function(Closure $next, $passable) {
        echo 'Second' . PHP_EOL;
        $passable ++; // +1
        return $next($passable);
    },

    function(Closure $next, $passable) {
        echo 'LastPipe' . PHP_EOL;
        $passable ++; // +1
        return $next($passable);
    }
];

// 反转
$pipes = array_reverse($pipes);

// http://www.php.net/manual/en/function.array-reduce.php
$pipeline = array_reduce($pipes, 
    // $stack 表示上一次跌代里的值，第一次为 $firstSlice 的值
    // $pipe 本次迭代的值
    function ($stack, $pipe) {
        return function ($passable) use ($stack, $pipe) {
            return call_user_func($pipe, $stack, $passable);
        };
    }, 
$firstSlice);

$passable = 1;
$result = call_user_func($pipeline, $passable);

print_r($result);// 1+1+1+1+6 = 10

/**
First
Second
LastPipe
The end
1
*/
```

最好的理解方式打印 `$pipeline`,得到如下的结构：

```
Closure Object
(
    [static] => Array
        (
            [stack] => Closure Object
                (
                    [static] => Array
                        (
                            [stack] => Closure Object
                                (
                                    [static] => Array
                                        (
                                            [stack] => Closure Object
                                                (
                                                    [static] => Array
                                                        (
                                                            [end] => Closure Object
                                                                (
                                                                    [parameter] => Array
                                                                        (
                                                                            [$passable] => <required>
                                                                        )

                                                                )

                                                        )

                                                    [parameter] => Array
                                                        (
                                                            [$passable] => <required>
                                                        )

                                                )

                                            [pipe] => Closure Object
                                                (
                                                    [parameter] => Array
                                                        (
                                                            [$next] => <required>
                                                            [$passable] => <required>
                                                        )

                                                )

                                        )

                                    [parameter] => Array
                                        (
                                            [$passable] => <required>
                                        )

                                )

                            [pipe] => Closure Object
                                (
                                    [parameter] => Array
                                        (
                                            [$next] => <required>
                                            [$passable] => <required>
                                        )

                                )

                        )

                    [parameter] => Array
                        (
                            [$passable] => <required>
                        )

                )

            [pipe] => Closure Object
                (
                    [parameter] => Array
                        (
                            [$next] => <required>
                            [$passable] => <required>
                        )

                )

        )

    [parameter] => Array
        (
            [$passable] => <required>
        )

)
```

使用  `call_user_func($pipeline, $passable)` 最开始执行的就是这个 $pipes[0]，然后逐个向后迭代。

## 设计优化

Laravel 这种实现方法，分析源代码还是蛮头痛的，理解起来还是有点费劲，它是一种逆向思维。当你理解的时候，你会发现本质实现上就是一个迭代器，我们可以用迭代器正向思维来实现它。

Laravel 要求你的每一个 迭代器都需要 `return $next($passable)` 用于返回下次代码的闭包，其实我们可以实现不用 `return` 也可以迭代，而且可以在 `$next` 添加更多的业务，将下一次迭代器的结果返回给当前的迭代器。

```php
<?php

$pipes = [
    function(Closure $next, $passable, $passable2) {
        echo 'First' . PHP_EOL;
        $passable ++; // +1
        $next($passable, $passable2);
        return 'The last return data is from the first';
    },

    function(Closure $next, $passable, $passable2) {
        echo 'Second' . PHP_EOL;
        $passable ++; // +1
        $next($passable, $passable2);
    },

    function(Closure $next, $passable, $passable2) {
        echo 'LastPipe' . PHP_EOL;
        $passable ++; // +1
        $nextReturn = $next($passable, $passable2);
        print_r($nextReturn);
    },

    // the end
    function(Closure $next, $passable, $passable2) {
        echo 'The end' . PHP_EOL;
        $passable += 6; // +6
        // 结果可以给上一层数据
        return [$passable, $passable2];
    },
];

// 生成迭代器遍历
// [ɪtə'reɪtə]
$iterators = call_user_func(
    function(array $pipe) {
        array_unshift($pipe, null);
        
        foreach ($pipe as $item) {
           // 利用生成器生成迭代器
           yield $item;
        }
    },

    $pipes
);

// 遍历迭代器
function traverse() {
    // 面向对象不使用 global
    // $this->iterators
    global $iterators;

    if(! $iterators->valid() || $iterators->next() || ! $iterators->valid()) {
        return;
    }

    $args = func_get_args();

    array_unshift($args, function() {
        return traverse(...func_get_args());
    });

    //老版本不支持 return $iterators->current()(...$args);
    $current = $iterators->current();
    return $current(...$args);
}

$passable = 1;
$passable2 = 6;

// 更多参数
$passmany = [$passable, $passable2];

$result = traverse(...$passmany);

print_r($result);

/*
First
Second
LastPipe
The end
Array
(
    [0] => 10
    [1] => 6
)
The last return data is from the first
*/
```

我们采用了正向思维来遍历我们需要执行的中间件代码，我们可以无需 `return $next` 就可以完成整个中间件业务迭代。如果你需要返回数据，也可以 return 数据回去，从最后一个逐个向第一个传递。实现过程有所不同，但是思想差不读，大家可以仔细体会一下。基于这个思想编写的管道组件，有需要的同学可以去看看。

[Leevel\Pipeline\Pipeline](https://github.com/hunzhiwange/framework/blob/master/src/Queryyetsimple/Pipeline/Pipeline.php)

## 参考文章

* [PHP闭包Closure与array_reduce结合的一个范例](https://www.cnblogs.com/springwind2006/p/7768493.html)
* [深入理解Laravel中间件](https://segmentfault.com/a/1190000007715254)
* [PHP array_reduce](http://www.php.net/manual/en/function.array-reduce.php)