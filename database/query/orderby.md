# 查询语言.orderBy

## 函数原型

``` php
public function order($mixExpr);
```

 > 说明：参数支持字符串以及它们构成的一维数组。

## 用法如下

``` php
# SELECT `test`.`tid` AS `id`,`test`.`tname` AS `value`  FROM `test`  ORDER BY `test`.`id` DESC,`test`.`name` ASC
database::table('test', 'tid as id,tname as value')->

orderBy('id DESC')->

orderBy('name')->

getAll();
    
# SELECT `test`.`tid` AS `id`,`test`.`tname` AS `value`  FROM `test`  ORDER BY `test`.`id` DESC
database::table('test', 'tid as id,tname as value')->

orderBy('test.id DESC')->

getAll();

# SELECT `test`.`tid` AS `id`,`test`.`tname` AS `value`  FROM `test`  ORDER BY SUM(`test`.`num`) ASC
database::table('test', 'tid as id,tname as value')->

orderBy('{SUM([num]) ASC}')->

getAll();
   
# SELECT `test`.`tid` AS `id`,`test`.`tname` AS `value`  FROM `test`  ORDER BY `test`.`title` ASC,`test`.`id` ASC,concat('1234',`test`.`id`,'ttt') DESC
database::table('test', 'tid as id,tname as value')->

orderBy("title,id,{concat('1234',[id],'ttt') desc}")->

getAll();
    
# SELECT `test`.`tid` AS `id`,`test`.`tname` AS `value`  FROM `test`  ORDER BY `test`.`title` ASC,`test`.`id` ASC,`test`.`ttt` ASC,`test`.`value` DESC
database::table('test', 'tid as id,tname as value')->

orderBy(['title,id,ttt', 'value desc'])->

getAll();

# SELECT `test`.`tid` AS `id`,`test`.`tname` AS `value`  FROM `test`  ORDER BY `test`.`title` DESC,`test`.`id` DESC,`test`.`ttt` ASC,`test`.`value` DESC
database::table('test', 'tid as id,tname as value')->

orderBy(['title,id,ttt asc', 'value'], 'desc')->

getAll();
```

## 快捷排序 latest/oldest

``` php
# SELECT `test`.* FROM `test` ORDER BY `test`.`create_at` DESC
database::->table('test')->

latest()->

getAll();

# SELECT `test`.* FROM `test` ORDER BY `test`.`foo` DESC
database::->table('test')->

latest('foo')->

getAll();

# SELECT `test`.* FROM `test` ORDER BY `test`.`create_at` ASC
database::->table('test')->

oldest()->

getAll();

# SELECT `test`.* FROM `test` ORDER BY `test`.`bar` ASC
database::->table('test')->

oldest('bar')->

getAll();
```