# Lists 循环

lists 标签主要用于在模板中循环输出数据集或者多维数组。

## 普通输出

然后我们在模版中定义如下代码，循环输出名言标题和 IT 名人：

``` html
<lists name="list" id="vo">
    {$vo.title}  {$vo.people}
</lists>
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $index = 0;
    $tmp = $list;
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$index;
            $mod = $index % 2; ?>
    <?php echo $vo->title; ?>  <?php echo $vo->people; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

<p class="tip">list 标签的 **name** 属性表示模板赋值的变量名称，因此不可随意在模板文件中改变。**id** 表示当前的循环变量，可以随意指定，但确保不要和 name 属性冲突。</p>

## 部分输出

支持输出部分数据，例如输出其中的第 **2～4** 条记录

``` html
<lists name="list" id="vo" offset="2" length='4'>
    {$vo.title} {$vo.people}
</lists>
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $index = 0;
    $tmp = array_slice($list, 2, 4);
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$index;
            $mod = $index % 2; ?>
    <?php echo $vo->title; ?> <?php echo $vo->people; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

## 输出偶数记录

lists 还支持按偶数输出，例如，我们在模板中写下如下的代码：   

``` html
<lists name="list" id="vo" mod="2">
    <?php if ($mod == 1): ?>
        {$vo.title} {$vo.people}
    <?php endif; ?>
</lists>
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $index = 0;
    $tmp = $list;
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$index;
            $mod = $index % 2; ?>
    <?php if ($mod == 1): ?>
        <?php echo $vo->title; ?> <?php echo $vo->people; ?>
    <?php endif; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

## 输出奇数记录

lists 还支持奇数记录的输出：

``` html
<lists name="arrList" id="arrVo" mod="2">
    <?php if ($mod == 0) : ?>
        {$arrVo.title} {$arrVo.people}
    <?php endif; ?>
</lists>
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $index = 0;
    $tmp = $list;
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$index;
            $mod = $index % 2; ?>
    <?php if ($mod == 0): ?>
        <?php echo $vo->title; ?> <?php echo $vo->people; ?>
    <?php endif; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

<p class="tip">奇数记录和偶数记录规定如下，我们以数组的 0 为开始，0、2、4为偶记录，其它的都为基数记录。</p>

## 控制换行

mod 属性还用于控制一定记录的换行，例如：

``` html
<lists name="arrList" id="arrVo" mod="2">
    {$arrVo.title} {$arrVo.people}
    <?php if( $mod == 0 ) : ?>
        <br>
    <?php endif; ?>
</lists>
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $index = 0;
    $tmp = $list;
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$index;
            $mod = $index % 2; ?>
    <?php echo $vo->title; ?> <?php echo $vo->people; ?>
    <?php if ($mod == 0): ?>
        <br>
    <?php endif; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

## 输出循环变量

我们在模版中写下如下的代码:

``` html
<lists name="arrList" id="vo" index="k">
    {$k} {$vo.people}
</lists>
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $k = 0;
    $tmp = $list;
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$k;
            $mod = $k % 2; ?>
    <?php echo $k; ?> <?php echo $vo->people; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

## 输出数组的索引 

如果要输出数组的索引，可以直接使用 **key** 变量，和循环变量不同的是，这个 key 是由数据本身决定，而不是循环控制的，这个 key 可以通过 key 属性指定。

例如：

``` html
<lists name="arrList" id="vo">     
    key: {$key}
</lists>  
```

模板编译后的结果：

``` php
<?php if (is_array($list)):
    $index = 0;
    $tmp = $list;
    if (count($tmp) == 0):
        echo "";
    else:
        foreach ($tmp as $key => $vo):
            ++$index;
            $mod = $index % 2; ?>
    key: <?php echo $key; ?>
        <?php endforeach;
    endif;
else:
    echo "";
endif; ?>
```

<p class="tip">大家看到没有，索引从 0 开始，而不是从 1 开始，这个是数组原始索引。</p>
