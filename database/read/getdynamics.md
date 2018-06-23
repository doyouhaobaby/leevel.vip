# 动态查询.getStart.getBy.getAllBy

## 支持 get10start2

``` php
# SELECT `test`.* FROM `test` LIMIT 3,10 
database::table('test')->

get10start3();
```

## 支持 getBy

``` php
# SELECT `test`.* FROM `test` WHERE `test`.`user_name` = '1111' LIMIT 1
database::table('test')->

getByUserName('1111');

# SELECT `test`.* FROM `test` WHERE `test`.`UserName` = '1111' LIMIT 1
database::table('test')->

getByUserName_('1111');
```

## 支持 getAllBy

``` php
# SELECT `test`.* FROM `test` WHERE `test`.`user_name` = '1111' AND `test`.`sex` = '222'
database::table('test')->

getAllByUserNameAndSex('1111', '222');

# SELECT `test`.* FROM `test` WHERE `test`.`UserName` = '1111' AND `test`.`Sex` = '222'
database::table('test')->

getAllByUserNameAndSex_('1111', '222');
```
