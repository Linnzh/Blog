# 理解路由

## Reference

- [Slim 4: Routing](http://www.slimframework.com/docs/v4/objects/routing.html)
- [Slim 4 - Tutorial](https://odan.github.io/2019/11/05/slim4-tutorial.html)

------------------------------------------------------------------------------------------------------------------------

## 前情

之前想学习框架，但始终对框架中的路由、模型、控制器、视图等概念搞到头大。之前终于理解了一点**依赖注入**，现在来了解一下路由（Route）

## 初始化项目

使用`composer`初始化项目：

```bash
composer init
```

引入`Slim`框架：

```bash
# 引入 Slim 框架
composer require slim/slim:"4.*"

# 引入 PSR-7 实现和 ServerRequest 构造
composer require slim/psr7

# 引入 PHP-DI 容器注入依赖
composer require php-di/php-di

# 再引入测试框架 PHPUnit
composer require --dev phpunit/phpunit
```

也可以直接在项目根目录新建`composer.json`文件，然后输入以下内容：

```json
{
    "require": {
        "slim/slim": "4.*",
        "slim/psr7": "^0.6.0",
        "php-di/php-di": "^6.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^8.5"
    },
    "autoload": {
        "psr-4": {
            "Linnzh\\": "src"
        }
    }
}
```

当然，上面的信息都可以更改，只需要确定`require`部分即可。

## 第一个路由

我们在`public`目录下新建一个名为`index.php`的入口文件。访问我们的本地服务器时将会运行该文件的内容。

按照官网写的案例，我们写下下面这些代码：

```php
<?php

require dirname(__FILE__, 2).'/vendor/autoload.php';

use Slim\Factory\Appfactory;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

// 创建 APP
$app = Appfactory::create();

// 第一个路由
$app->get('/', function (Request $request, Response $response) {
    $response->getBody()->write('Hello, World!');
    return $response;
});

// 运行 APP 实例
$app->run();
```

主要步骤就是：

1. 引入 composer 的自动加载文件
    - `require dirname(__FILE__, 2).'/vendor/autoload.php';`
2. 创建应用实例`$app`
    - `Appfactory::create()`
3. 注册一个`get`方法的路由
    - 这里的路由解析为：访问网站根目录时，网站响应并输出`Hello, World!`
4. 运行 APP 实例

当然也不要忘记使用`use`。

在控制台输入：`php -S localhost:8080 -t public`，然后在浏览器中访问 <localhost:8080>，即可看到输出`Hello, World!`！

### 关于路由

很简单地，我们创建了一个路由。那么路由有哪些需要注意的地方呢？

详细说明可参照[官网](http://www.slimframework.com/docs/v4/objects/routing.html)。要看总结就继续：

1. 路由是什么

是网络资源定位符 **URL** 对 HTTP 响应的处理程序。它负责给不同的 URL 分配对应的处理程序。

例如在“第一个路由”中定义的路由，它实际上是将 URL （这里是 <localhost:8080> ）或者符合一定正则模式的 URL 分配给一个闭包函数处理，在闭包函数中接受三个参数（第三个参数可选，案例中没写，官网有说明）：请求（Request）、响应（Response）和路径中的命名参数（args）。在函数体中进行相关处理，最后返回一个**响应**（Response）。响应就是我们在浏览器中看到的所有内容了。

关于注册路由的详细信息可查看[官网](http://www.slimframework.com/docs/v4/objects/routing.html)说明。这里给出几个小示例：

```php
/**
 * http://localhost:8080/test
 * 
 * 一个 JSON 格式化后的数组
 * 
 * 重点：$response->withHeader('Content-Type', 'application/json')
 * 设置了响应头信息
 */
$app->get('/test', function (Request $request, Response $response) {
    $response->getBody()->write(json_encode(['name'=>'Linnzh', 'github'=>'https://github.com/Linnzh'], JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT));
    return $response->withHeader('Content-Type', 'application/json');
});


/**
 * http://localhost:8080/example
 * http://localhost:8080/example/linnzh
 * http://localhost:8080/example/linnzh/is/so/cute
 * 
 * 使用了一部分路由正则规则：方括号->可选；花括号->命名参数；.* ->任意字符串
 */
$app->get('/example[/{params:.*}]', function (Request $request, Response $response, $args) {
    $params = explode('/', $args['params']);

    $response->getBody()->write(json_encode($params));
    return $response->withHeader('Content-Type', 'application/json');
});


/**
 * http://localhost:8080/books/list
 * http://localhost:8080/books/list/2019
 * http://localhost:8080/books/list/2019/12
 * 
 * 使用命名路由：setName()
 */
$app->get('/books/list[/{year:[0-9]+}[/{month:[0-9]+}]]', function (Request $request, Response $response, $args) {
    if($args) {
        $response->getBody()->write('您正在查看'.$args['year'].'年的书籍列表');
    } else {
        $response->getBody()->write('您正在查看书籍列表');
    }
    return $response;
})->setName('list'); // 给路由命名
// var_dump($app->getRouteCollector()->getRouteParser()->urlFor('list', ['year','month'], ['year'=>'2019','month'=>12]));// 使用命名路由访问

/**
 * http://localhost:8080/src
 * http://localhost:8080/src/controllers/UserController.php
 * 
 * 重定向路由。这里将符合某种正则模式的路由（例如不存在的网址）重定向至首页
 * 注意不能让已定义的路由规则符合重定向规则
 */
$app->redirect('/src[/{params:.*}]', '/', 301);

```

2. 路由支持哪些 HTTP 方法

从官方文档说明可以得知，路由支持以下几种 HTTP 方法：

- GET
- POST
- PUT
- DELETE
- OPTIONS
- PATHCH
- Any
- Custom

注册路由时一律按照以下格式：

```php
$app->get('/url', function(Request $request, Response $response, $args) {
    // code

    return $response;
});
```

即使用 APP 实例调用相关 HTTP 方法（小写），并传入两个参数：路由模式（Route Pattern）和回调函数（Callback）。回调函数必须返回`ResponseInterface`接口实例。

笔者常用的是`GET`方法，配合前端数据，进行一些处理并返回数据。`POST`可以用来处理文件和隐私数据。其他 HTTP 方法并没有用过。以后接触到再说明。

3. 更多路由注册方式

Slim 框架还支持以下路由特征：

- 路由分组（Route groups）
- 路由中间件（Route middleware）
- 路由表达式缓存（Route expressions caching）
- 依赖注入容器路由解析（Container Resolution）

这些方式笔者还没有尝试，一些概念例如中间件（Middleware）、控制器（Controller）、视图（View）还并不明白。等理解了再另开说明。


### routes.php

项目变得庞大时，在入口文件`index.php`中定义所有的路由可能会比较繁琐且不清晰。因此我们可以将路由拆开，在文件中定义，然后在入口文件引入。例如我们可以在`/app/routes.php`中定义路由：

```php
<?php
declare(strict_types=1);

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\App;

return function (App $app) {
    $app->get('/test', function (Request $request, Response $response) {
        $response->getBody()->write(json_encode(['name'=>'Linnzh', 'github'=>'https://github.com/Linnzh'], JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT));
        return $response->withHeader('Content-Type', 'application/json');
    });
    $app->get('/example[/{params:.*}]', function (Request $request, Response $response, $args) {
        $params = explode('/', $args['params']);
    
        $response->getBody()->write(json_encode($params));
        return $response->withHeader('Content-Type', 'application/json');
    });
    $app->get('/books/list[/{year:[0-9]+}[/{month:[0-9]+}]]', function (Request $request, Response $response, $args) {
        if($args) {
            $response->getBody()->write('您正在查看'.$args['year'].'年的书籍列表');
        } else {
            $response->getBody()->write('您正在查看书籍列表');
        }
        return $response;
    })->setName('list'); // 给路由命名
    
    $app->redirect('/src[/{params:.*}]', '/', 301);
};
```

然后在`/public/index.php`中添加：

```php
<?php

require '../vendor/autoload.php';

use Slim\Factory\Appfactory;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

// 创建 APP
$app = Appfactory::create();

// 注册路由
$routes = require __DIR__ . '/../app/routes.php';
$routes($app);

$app->run();
```

引入多个路由文件都是可以的。

### 路由分组（Route groups）

路由分组有利于逻辑整理。组路由可以嵌套子路由，并且子路由可以使用组路由中的命名占位符。例如（文件`/app/routes/user.php`）：

```php
<?php
declare(strict_types=1);

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\App;
use Slim\Routing\RouteCollectorProxy;
use Slim\Routing\RouteGroup;

return function (App $app) {
    $app->group('/user/{id:[0-9]+}', function (RouteCollectorProxy $group) {
        $group->get('/reset-password', function (Request $request, Response $response) {
            // http://tmp/user/10/reset-password
            // 重置密码操作
            $response->getBody()->write('重置密码成功！');
            return $response;
        })->setName('user-password-reset');

        $group->get('/comments', function (Request $request, Response $response) {
            // http://tmp/user/10/comments
            // 获取所有发出的评论
            $response->getBody()->write('获取所有发出的评论成功！');
            return $response;
        })->setName('user-comments');

        $group->map(['GET', 'DELETE', 'PATCH', 'PUT'], '/list', function (Request $request, Response $response, $args) {
            // 对应 HTTP 操作
            return $response;
        })->setName('user-list');

        // 在组闭包内部，闭包函数将被绑定到容器实例。
        // $this 表示 接口 Psr\Container\ContainerInterface 的实例
        $group->get('', function (Request $request, Response $response, $args) {
            // http://tmp/user/5
            // 获取用户 ID 为指定ID 的信息
            $stmt = $this->get('db')->prepare('SELECT `name`,`profile` FROM `user` WHERE `id` = ?');
            $stmt->bindValue(1, $args['id']);
            $stmt->execute();
            $response->getBody()->write(json_encode($stmt->fetch(PDO::FETCH_ASSOC), JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE));
            return $response->withHeader('Content-Type', 'application/json');
        });
    });
};
```

上述组路由中最后定义的路由中，使用了`$this`变量。该变量表示一个接口`Psr\Container\ContainerInterface`的实例，表现为 APP 实例的容器实例。
而容器实例必须在 APP 创建之前定义。如下：

```php
<?php

require dirname(__FILE__, 2) . '/vendor/autoload.php';

use Slim\Factory\Appfactory;

use DI\Container;

// 创建一个容器
$container = new Container();
AppFactory::setContainer($container);

// 创建 APP
$app = Appfactory::create();

// 注册路由
$routes = require dirname(__FILE__, 2) . '/app/routes.php';
$routes($app);
$routes = require dirname(__FILE__, 2) . '/app/routes/user.php';
$routes($app);

// 载入容器依赖
$dependencies = require dirname(__FILE__, 2) . '/app/dependencies.php';
$dependencies($app);

// 运行 APP 实例
$app->run();

```

而容器依赖可以这样定义（文件`/app/dependencies.php`）：

```php
<?php

declare(strict_types=1);

use Slim\App;

return function (App $app) {
    // 获取容器实例
    $container = $app->getContainer();

    // 添加容器服务
    $container->set('db', function () {
        $dbconfig = [
            'driver' => 'mysql',
            'host' => 'localhost',
            'port' => '3306',
            'dbname' => 'test',
            'username' => 'root',
            'passwd' => 'password',
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
        ];
        $dsn = sprintf('%s:host=%s;port=%d;dbname=%s', $dbconfig['driver'], $dbconfig['host'], $dbconfig['port'], $dbconfig['dbname']);
        $options = [
            // Turn off persistent connections
            PDO::ATTR_PERSISTENT => false,
            // Enable exceptions
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            // Emulate prepared statements
            PDO::ATTR_EMULATE_PREPARES => false,
            // Set default fetch mode to array
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            // Set character set
            PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci'
        ];
        return new PDO($dsn, $dbconfig['username'], $dbconfig['passwd'], $options);
    });
};

```

> 注：关于容器依赖注入可以看[这里](/posts/understand-dependency-injection.md)

使用容器服务时只需要像这样：`$app->get('serviceName')`。

当然，关于容器服务还有更复杂的配置。以后用到再说。

### 依赖注入容器路由解析（Container Resolution）

路由注册并不限于定义一个闭包函数，并且闭包函数的使用代价比较昂贵（每次请求就要重新创建），可以换成**将字符串解析为函数调用**。

1. 例如，我们先建立一个`HomeController`类，并在里面添加一个`welcome()`方法：

```php
<?php

namespace Linnzh\Controllers;

use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

class HomeController
{
    public function welcome(Request $request, Response $response, $args)
    {
        $response->getBody()->write('welcome');
        return $response;
    }

}
```

然后我们可以在`/app/routes/route.php`中注册一个访问该类`welcome()`方法的路由：

```php

$app->get('/', HomeController::class.':welcome');

```

这看起来有点麻烦，还需要拼接字符串。但是该方式适合一些耦合较高的操作。

2. 另一种方式是省去拼接字符串的功夫，直接定义对应操作的类文件。也就是说，我们不指定方法，而是指定类。

这种方式适用于零散的、耦合度不高的一些操作。

例如，预期：访问主页，并显示欢迎语

```php

$app->get('/', WelcomeAction::class);

```

我们需要新建一个`WelcomeAction.php`类文件：

```php
<?php

namespace Linnzh\Actions;

use Psr\Http\Message\RequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Container\ContainerInterface;

class WelcomeAction
{
    protected $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function __invoke($request, $response, $args)
    {
        $response->getBody()->write('Welcome');
        return $response;
    }
}
```

这个类必须实现`__invoke()`魔术方法。在试图以调用函数的形式调用一个对象时，该魔术方法被自动调用。
