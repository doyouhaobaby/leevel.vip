# 基于Zephir构建快速的PHP扩展框架或组件

![Zephir console](../_media/zephir-logo.png)

Zephir 是一个开源的，可以用高级语言安全快速地编写 PHP 的 C 扩展, 2015 年 4 月 Phalcon 2.0 已经改用 Zephir 重写。

Phalcon 是开源、全功能栈、使用 C 扩展编写、针对高性能优化的 PHP 5 框架。Phalcon 和 Yaf 是 C 类扩展开发框架的代表。


## 构建 dhb 框架项目

今天我们基于 Zephir 快速地实现一个简单的 dhb 开发框架来展示一下这门语言的魅力。

这里安装环境我就不写了，大家自行学习搭建，这是第一步，也是重要的一步。

### 当前开发环境

 * php-7.1.6
 * gcc version 5.4.0 20160609
 * Ubuntu 16.04.2 LTS
 * zephir-0.10.7


![Zephir console](../_media/zephir-console.png)

### 创建一个 dhb 框架

```
zephir init dhb
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb# ls
dhb
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb# cd dhb
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/dhb# ls
config.json  dhb  ext
```


> dhb 是源代码目录，ext 是扩展目录，扩展既可以采用 zephir 提供的命令行工具来安装，也可以进入 ext 目录进行编译。

```
$/path/to/phpize
$./configure --with-php-config=/path/to/php-config
$make && make install
```

### 项目配置文件

系统会自动创建一个标准的 zephir 项目，其中 config.json 就是这个项目的配置，相当于 Composer 的 composer.json 和 npm 中的 package.json，其中最重要的是命名空间 `namespace`, 项目遵循 `Psr4` 规则，代码风格推荐使用 `Psr2`。

```
    "namespace": "dhb",
    "name": "dhb",
    "description": "The framework for dhb.",
    "author": "rsung",
    "version": "0.0.1",
```


## 构建框架结构

为了更好地搭建整个框架，我们需要调整一下整个项目的目录结构，初始化两个 composer 项目，一个 dhb/app,一个框架 dhb/framework。

![Zephir console](../_media/dhb-framework.png)

### dhb/app 配置文件 

位于 `/data/codes/dhb/composer.json`

```
{
    "name": "dhb/app",
    "description": "dhb app",
    "authors": [
        {
            "name": "rsung.com",
            "email": "test@qq.com"
        }
    ],
    "require": {}
}
```

### dhb/framework 配置文件 

位于 `/data/codes/dhb/vendor/dhb/framework/composer.json`

```
{
    "name": "dhb/framework",
    "description": "dhb framework",
    "authors": [
        {
            "name": "rsung",
            "email": "test@rsung.com"
        }
    ],
    "require": {}
}
```

## 搭建 phpunit 测试框架

基础组件需要经过单元测试覆盖才能够保证代码质量，这里我们扩展部分的单元测试也使用 phpunit,我们不使用 phpt 那种测试方式.

修改框架的 composer, 安装两个包，phpunit/phpunit 和 nunomaduro/collision，第一个是测试框架，第二个友好地调试 phpunit 异常。

```
{
    "name": "dhb/framework",
    "description": "dhb framework",
    "authors": [
        {
            "name": "rsung",
            "email": "test@rsung.com"
        }
    ],
    "require": {
        "php": "^7.1.3",
        "nunomaduro/collision": "~2.0"
    },
    "require-dev": {
        "phpunit/phpunit": "~7.0"
    }
}
```

### 安装好 composer install，然后添加 phpunit.xml 和 tests 目录，新建 bootstrap.php 启动文件。

位于 `/data/codes/dhb/vendor/dhb/framework/phpunit.xml`


```
<?xml version="1.0" encoding="UTF-8"?>
<phpunit
    backupGlobals="false"
    backupStaticAttributes="false"
    bootstrap="tests/bootstrap.php"
    colors="true"
    convertErrorsToExceptions="true"
    convertNoticesToExceptions="true"
    convertWarningsToExceptions="true"
    processIsolation="false"
    stopOnFailure="false"
    syntaxCheck="false">
    <listeners>
        <listener class="NunoMaduro\Collision\Adapters\Phpunit\Listener" />
    </listeners>
    <testsuites>
        <testsuite name="Application Test Suite">
            <directory>./tests/</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist>
            <directory suffix=".php">./</directory>
            <exclude>
                <directory suffix=".php">tests</directory>
                <directory suffix=".php">vendor</directory>
            </exclude>
        </whitelist>
    </filter>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="QUEUE_DRIVER" value="sync"/>
    </php>
</phpunit>
```

位于 `/data/codes/dhb/vendor/dhb/framework/tests/bootstrap.php`

```php
<?php declare(strict_types=1);

error_reporting(-1);

ini_set('xdebug.max_nesting_level', '200');

ini_set('memory_limit', '512M');

$vendorDir = __DIR__ . '/../vendor';

if (false === is_file($vendorDir . '/autoload.php')) {
    throw new Exception('You must set up the project dependencies, run the following commands:
        wget http://getcomposer.org/composer.phar
        php composer.phar install');
} else {
    include $vendorDir . '/autoload.php';
}

spl_autoload_register(function($class) {
    if (0 === stripos($class, 'Tests\\')) {
        $path = __DIR__ . '/' . strtr(substr($class, 5), '\\', '/') . '.php';
        
        if (is_file($path) === true && is_readable($path) === true) {
            require_once $path;

            return true;
        }
    }
});

```

### 编写一个简单的单元测试文件看看 phpunit 是否正常工作

位于 `/data/codes/dhb/vendor/dhb/framework/tests/HelloTest.php`


```php 
<?php declare(strict_types=1);

namespace Tests;

use PHPUnit\Framework\TestCase;

class HelloTest extends TestCase
{

    public function testFoo()
    {
        $this->assertEquals('foo', 'foo');
    }
}

```

使用 php vendor/bin/phpunit tests/HelloTest.php 运行一下

![Zephir console](../_media/phpunit-hello.png)

## 编写第一个扩展类

我们从一个经典的 hello world 开始我们的项目，并且搭建整个扩展开发的环境。

### 编写一个 hello 的组件

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/hello/world.zep`

```
namespace Dhb\Hello;

class World
{
    public function welcome() -> string 
    {
        return "hello dhb framework";
    }
}
```

进入扩展目录，编译并且安装

```
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/vendor/dhb/framework/c# zephir build
make: Warning: File 'clean' has modification time 622 s in the future
make: warning:  Clock skew detected.  Your build may be incomplete.
Preparing for PHP compilation...
Preparing configuration file...
Compiling...
Installing...
Extension installed!
Add extension=dhb.so to your php.ini
Don't forget to restart your web server
```

### PHP 环境加入 dhb 扩展

修改 php.ini 加入 extendsion = dhb.so

![Zephir console](../_media/phpini-dhb-so.png)

使用 php -m 看看扩展是否安装,如你所见一切 OK

![Zephir console](../_media/phpm-dhb-so.png)


### 编写 hello 组件的单元测试

开发组件的时候，单元测试即是测试调试我们开发，不会有重启 php-fpm 这种操作。

编写一个简单的单元测试

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Hello/WorldTest.php`

```
<?php declare(strict_types=1);

namespace Tests\Hello;

use PHPUnit\Framework\TestCase;
use Dhb\Hello\World;

class WorldTest extends TestCase
{

    public function testWelcome()
    {
        $this->assertEquals('hello dhb framework', (new World)->welcome());
    }
}

```


运行单元测试文件，测试成功

```
php vendor/bin/phpunit tests/Hello/WorldTest.php 
PHPUnit 7.1.5 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 345 ms, Memory: 4.00MB

OK (1 test, 1 assertion)
```

### 编译说明

todo


## 开始造框架基础组件之 IOC 容器

编写一个 zephir 版本的 IoC 容器，用于管理框架服务，实现 PHP 依赖注入。容器实现比较复杂，这里造好了，有兴趣的去看一下。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/di` 目录,包含 4 个文件

```
container.zep 容器
container.zep 容器接口
normalizeexception.zep 容器异常
provider.zep 容器服务提供者
```

为容器编写单元测试文件

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Di/ContainerTest.php`

运行单元测试文件，测试成功

```
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/vendor/dhb/framework# php vendor/bin/phpunit tests/Di/ContainerTest.php 
PHPUnit 7.1.5 by Sebastian Bergmann and contributors.

.......................                                           23 / 23 (100%)

Time: 270 ms, Memory: 4.00MB

OK (23 tests, 62 assertions)
```

## 开始造框架基础组件之配置

应用程序需要配置来封装服务，这个时候需要一个基本用于管理基本配置文件的组件。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/option` 目录,包含 2 个文件

```
option.zep 配置文件
ioption.zep 配置接口
```

为配置编写单元测试文件

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Option/OptionTest.php`

运行单元测试文件，测试成功

```
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/vendor/dhb/framework# php vendor/bin/phpunit tests/Option/OptionTest.php
PHPUnit 7.1.5 by Sebastian Bergmann and contributors.

.......                                                             7 / 7 (100%)

Time: 249 ms, Memory: 4.00MB

OK (7 tests, 46 assertions)
```

## 开始造框架基础组件之日志组件

框架需要自己的日志组件，记录系统运行的错误。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/log` 目录,包含 2 个文件

```
log.zep 日志文件
ilog.zep 日志接口
```

为日志组件编写单元测试文件

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Log/LogTest.php`

运行单元测试文件，测试成功

```
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/vendor/dhb/framework# php vendor/bin/phpunit tests/Log
PHPUnit 7.1.5 by Sebastian Bergmann and contributors.

..                                                                  2 / 2 (100%)

Time: 250 ms, Memory: 4.00MB

OK (2 tests, 3 assertions)
```


## 开始造框架基础组件之 HTTP 组件

HTTP 组件是框架必须的基础组件，这里编写 HTTP 组件。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/http` 目录,包含 5 个文件

```
bag.zep 参数包装文件
headerbag.zep header 包装
jsonrepsonse.zep json 响应
request.zep 请求
response.zep 基础响应
```

为 HTTP 组件编写单元测试文件

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Http`

运行单元测试文件，测试成功

```
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/vendor/dhb/framework# php vendor/bin/phpunit tests/Http
PHPUnit 7.1.5 by Sebastian Bergmann and contributors.

.......................................                           39 / 39 (100%)

Time: 294 ms, Memory: 4.00MB

OK (39 tests, 88 assertions)
```

## 开始造框架基础组件之 Kernel 组件

传入 Request,返回 Response，这就是内核组件的责任。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/kernel` 目录,包含 2 个文件

```
kernel.zep 内核
headerbag.zep 内核接口
```

为 Kernel 组件编写单元测试文件

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Kernel/KernelTest.php`

运行单元测试文件，测试成功

```
root@vagrant-ubuntu-10-0-2-5:/data/codes/dhb/vendor/dhb/framework# php vendor/bin/phpunit tests/Kernel/KernelTest.php 
PHPUnit 7.1.5 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 235 ms, Memory: 4.00MB

OK (1 test, 1 assertion)
```

此时，我们 Kernel 组件还非常地弱小，测试一下。

```
namespace Dhb\Kernel;

use Dhb\Http\Request;
use Dhb\Http\Response;
use Dhb\Kernel\IKernel;

class Kernel implements IKernel
{
    /**
     * 响应 HTTP 请求
     *
     * @param \Dhb\Http\Request $request
     * @return \Dhb\Http\Response
     */
    public function handle(<Request> request)-> <Response>
    {
        return new Response("hello world");
    }
}
```

后续我们将继续扩展这个组件，我们开始在应用层接入框架来看看。


## 搭建应用

框架部分基础搭建完毕后面继续，这里开始去搭建应用部分。


### 搭建应用入口文件

位于 `/data/codes/dhb/www/index.php`,内容如下

```php
<?php declare(strict_types=1);

use Dhb\Http\Request;
use Dhb\Kernel\Ikernel;
use Dhb\Kernel\kernel;
use Dhb\Di\Container;
use Dhb\Di\IContainer;

// composer
require_once __DIR__ . '/../vendor/autoload.php';

// 注册基础服务
$container = Container::singletons();

$container->singleton('container', $container);

$container->alias('container', [
    IContainer::class,
    Container::class
]);

function app($server = null, array $args = [])
{
    if ($server === null) {
        return Container::singletons();
    } else {
        return Container::singletons()->make($server, $args);
    }
}

$container->singleton(IKernel::class, Kernel::class);

$request = Request::createFromGlobals();

$container->singleton('request', $request);

$container->alias('request', Request::class);

// 执行调度
$kernel = $container->make(IKernel::class);

$response = $kernel->handle($request);

$response->send();
```

为了方便访问，搭建 nginx 站点，指向我们的 `/data/codes/dhb/www`，并且配置一个 dhbframework.cn 的域名 host 一下:

```
server {
    add_header HostName chengdu-vm02;
    listen 8080;
    server_name dhbframework.cn;
    error_log  /var/log/nginx/q.error.log;
    access_log /var/log/nginx/q.access.log main;
    root /data/codes/dhb/www;
    index  index.html index.php;

    try_files $uri $uri/ @rewrite;

    location @rewrite {
        rewrite ^/(.*)$ /index.php?_url=/$1;
    }

    location ~ \.php$ {
        fastcgi_pass  127.0.0.1:9002;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ \.(md|json)$ {
        deny all;
    }

    location ~ /status$ {
        stub_status on;
        allow  all;
    }
}
```

重启 nginx 和 php-fpm，注意修改框架每次都需要重启 php-fpm，因为 PHP 扩展会常驻内存。

```
service nginx restart
service php716-fpm restart
```

![Zephir console](../_media/dhbframework-helloworld.png)


### 开始造框架基础组件之 路由组件

传入 Request,返回 Response，路由分发。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/router` 目录,包含 2 个文件

```
router.zep 路由
irouter.zep 路由接口
```

为 路由 组件编写单元测试文件

位于 `/data/codes/dhb/vendor/dhb/framework/tests/Router/RouterTest.php`

路由的单元测试时空的，后面自行可以完善，路由只提供了一个 dispach 方法来分发路由

```
/**
 * 分发请求到路由
 *
 * @param \Dhb\Http\Request $request
 * @return \Dhb\Http\Response
 */
public function dispatch(<Request> request);
```

### 入口文件注入路由到容器

位于 `/data/codes/dhb/www/index.php`

```php 
use Dhb\Router\Router;
use Dhb\Router\IRouter;

...

$container->singleton('router', new Router($container));

$container->alias('router', [
    IRouter::class,
    Router::class
]);
```

修改我们的 kernel ，搭建路由调度。

位于 `/data/codes/dhb/vendor/dhb/framework/c/dhb/kernel/kernel.zep`

```
namespace Dhb\Kernel;

use Dhb\Http\Request;
use Dhb\Http\Response;
use Dhb\Kernel\IKernel;
use Dhb\Router\IRouter;

class Kernel implements IKernel
{

    /**
     * 路由
     *
     * var \Dhb\Router\IRouter
     */ 
    protected router;

    /**
     * 构造函数
     *
     * @param \Dhb\Router\IRouter $router
     */
    public function __construct(<IRouter> router)
    {
        let this->router = router;
    }

    /**
     * 响应 HTTP 请求
     *
     * @param \Dhb\Http\Request $request
     * @return \Dhb\Http\Response
     */
    public function handle(<Request> request)-> <Response>
    {
        var response;

        let response = this->router->dispatch(request);

        return response;
    }
}
```

然后访问首页 `http://dhbframework.cn` 将报错

```html 
<br />
<b>Fatal error</b>:  Uncaught InvalidArgumentException: The node App\App\Controller\Home-&gt;handle() is not found. in /data/codes/dhb/www/index.php:51
Stack trace:
#0 [internal function]: Dhb\Router\Router-&gt;nodeNotFound()
#1 [internal function]: Dhb\Router\Router-&gt;matchRouter()
#2 [internal function]: Dhb\Router\Router-&gt;dispatch(Object(Dhb\Http\Request))
#3 /data/codes/dhb/www/index.php(51): Dhb\Kernel\Kernel-&gt;handle(Object(Dhb\Http\Request))
#4 {main}
  thrown in <b>/data/codes/dhb/www/index.php</b> on line <b>51</b><br />
```

然后访问例外一个地址 `http://dhbframework.cn/hello/world?id=1` 将报错

```html 
<br />
<b>Fatal error</b>:  Uncaught InvalidArgumentException: The node App\App\Controller\Hello-&gt;world() is not found. in /data/codes/dhb/www/index.php:51
Stack trace:
#0 [internal function]: Dhb\Router\Router-&gt;nodeNotFound()
#1 [internal function]: Dhb\Router\Router-&gt;matchRouter()
#2 [internal function]: Dhb\Router\Router-&gt;dispatch(Object(Dhb\Http\Request))
#3 /data/codes/dhb/www/index.php(51): Dhb\Kernel\Kernel-&gt;handle(Object(Dhb\Http\Request))
#4 {main}
  thrown in <b>/data/codes/dhb/www/index.php</b> on line <b>51</b><br />
```


我们的路由组件已经起作用了，因为没有接管异常所以系统抛出了异常，接下来编写控制器。

### 编写控制器

为应用添加一个 psr4，App 的命名空间。

位于 `/data/codes/dhb/composer.json`

```
"autoload": {
    "psr-4": {
        "App\\": "app/"
    }
}

...

composer dump-autoload -o
```

访问地址 `http://dhbframework.cn`
位于 `/data/codes/dhb/app/App/App/Controller/Home.php`

```php 
<?php declare(strict_types=1);

namespace App\App\Controller;

class Home
{

    public function handle()
    {
        return ['home' => 'world'];
    }
}
```

响应


```
{"home":"world"}
```


访问地址 `http://dhbframework.cn/hello/world?id=222`
位于 `/data/codes/dhb/app/App/App/Controller/Hello.php`

```php 
<?php declare(strict_types=1);

namespace App\App\Controller;

class Hello
{

    public function world()
    {
        return ['hello is' => app('request')->query->get('id', 'deafult')];
    }
}
```

响应


```
{"hello is":"222"}
```

访问地址 `http://dhbframework.cn/world/hello`
位于 `/data/codes/dhb/app/App/App/Controller/World/Hello.php`

```php 
<?php declare(strict_types=1);

namespace App\App\Controller\World;

class Hello
{

    public function handle()
    {
        return 'acton class with handle';
    }
}
```

响应


```
acton class with handle
```

访问地址 `http://dhbframework.cn/foo/bar/c/a`
位于 `/data/codes/dhb/app/App/App/Controller/Foo/Bar/C.php`

```
<?php declare(strict_types=1);

namespace App\App\Controller\Foo\Bar;

class C 
{

    public function a()
    {
        return 'Hello i am here App\App\Controller\Foo\Bar\A->c()';
    }
}
```

响应


```
Hello i am here App\App\Controller\Foo\Bar\A->c()
```

访问地址 `http://dhbframework.cn/goods/111/show`
位于 `/data/codes/dhb/app/App/App/Controller/Goods.php`

```
<?php declare(strict_types=1);

namespace App\App\Controller;

use Dhb\Http\Request;

class Goods
{

    public function show(Request $request)
    {
        return ['goods-id' => $request->params->get('_param0')];
    }
}
```

响应


```
{"goods-id":"111"}
```

## 框架接入日志服务

。。。
