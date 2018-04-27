# PHP 服务端渲染 VUE 之 php-v8js 扩展

## PHP 运行 Google v8js

为了能够在服务端渲染 vue，我们需要安装一个 php 扩展，安装方法请自行搜索，下面是 vue 官方的提供的 PHP 代码。

```php
<?php
$vue_source = file_get_contents('/path/to/vue.js');
$renderer_source = file_get_contents('/path/to/vue-server-renderer/basic.js');
$app_source = file_get_contents('/path/to/app.js');

$v8 = new V8Js();

$v8->executeString('var process = { env: { VUE_ENV: "server", NODE_ENV: "production" }}; this.global = { process: process };');
$v8->executeString($vue_source);
$v8->executeString($renderer_source);
$v8->executeString($app_source);
?>
```

## 前端 VUE 代码

这里的 app.js 代码如下，非常的简单。

```js
var vm = new Vue({
  template: `<div>{{ msg }}</div>`,
  data: {
    msg: 'hello'
  }
})

// exposed by vue-server-renderer/basic.js
renderVueComponentToString(vm, (err, res) => {
  print(res)
})
```

## 后记

这里这个可以运行其它 Javascript 代码，这里我也封装了一个执行 V8 的组件。

 [QueryPHP 之 View.V8](https://github.com/hunzhiwange/framework/blob/master/src/Queryyetsimple/View/V8.php)