# 理解依赖注入[译]

## Reference

- [Dependency Injection: Huh?](https://code.tutsplus.com/tutorials/dependency-injection-huh--net-26903)
- [PHP: The Right Way](https://phptherightway.com/)
- [Slim](http://www.slimframework.com/)
- [浅谈控制反转与依赖注入](https://zhuanlan.zhihu.com/p/33492169)

------------------------------------------------------------------------------------------------------------------------

## 前情

近来一直有想学习PHP框架的想法，也试着去找了找比较轻量级的框架，例如[Slim](http://www.slimframework.com/)，但在看文档的过程中始终很难理解一些概念，比如说*依赖注入*（Dependency Injection）。找了许多文章但还是一头雾水，不得其法。机缘巧合之下，克服了对英语的恐惧，借助 Google 翻译插件看完了[PHP: The Right Way](https://phptherightway.com/)，发现有一篇讲解依赖注入的文章：[Dependency Injection: Huh?](https://code.tutsplus.com/tutorials/dependency-injection-huh--net-26903)，通篇看下来，感觉浅显易懂，也没有各种复杂专业的词汇，至少能让人初步知道依赖注入到底是什么、干什么用的。

故此，借助这篇文章，来记录一点自己关于*依赖注入*的理解。

## 正文

先来看一个简单的示例：

> 这里假设我们已经创建了一个名为`SingleConnection.php`的数据库连接类，并将其放在了`Linnzh\Db`命名空间下。

```php
<?php

use Linnzh\Db\SingleConnection;

class Photo
{
    protected $conn;# 数据库连接实例

    public function __construct()
    {
        $this->conn = SingleConnection::getInstance();
    }
}
```

上面的代码好像没什么毛病：访问一个图片类（Photo），并在初始化构造对象时获取一个数据库连接。

但是。`__construct()`方法强制性让我们使用了`SingleConnection`这个类，也就是说：**该方法硬编码了一个依赖项**。一个图片类去做了连接数据库的事情，这违反了“**单一职责原则**”。

怼着数据库连接去想，我们需要连接MSSQL而不是MySQL怎么办？或许你会想修改`SingleConnection.php`的连接配置，或者更高级点，写一个数据库适配器类。但这样做将让`SingleConnection`类失去类的意义。或者我们不想要连接数据库怎么办？为什么一定要连接数据库呢？这违反了职责分离的概念。`Photo`类应该只专注于与图片直接有关的事情，而不是忙于其他没有直接关联的外部事件。

打个比方，闹钟在早上八点一十分叫你起床去上班，你去上班这件事，只能你自己去做，而不能让闹钟完成。闹钟只负责在八点一十叫醒你。这就是单一职责原则。

现在让我们来解决上面代码的两个问题：
- 写死了使用的数据库连接类`SingleConnection`
- 在对象初始化即连接数据库

第一个问题我们可以使用**构造器注入**方法来解决，这也是初步的**依赖注入雏形**。请看下方代码：

```php
<?php

namespace Linnzh\Model;

class Photo
{
    protected $conn;# 数据库连接实例

    /**
     * 构造函数
     *
     * @param mixed $conn  数据库连接实例
     */
    public function __construct($conn)
    {
        $this->conn = $conn;
    }
}
```

可以看到，我们将数据库连接实例作为一个参数传递给构造函数。使用时只需将需要的数据库连接传递给构造函数即可，避免了只能使用固定的`SingleConnection`类来进行数据库连接。使用如下：

```php
<?php

use Linnzh\Db\SingleConnection;
use Linnzh\Model\Photo;

$conn = SingleConnection::getInstance();
$photo = new Photo($conn);
```

第二个问题也很容易解决，给类添加一个`setDB()`方法即可，这也被称为**设置器注入**。它将构造函数的工作移交给一个类方法，使用者只需要在特定时刻调用该方法。实现如下：

```php
<?php

namespace Linnzh\Model;

class Photo
{
    protected $conn;# 数据库连接实例

    /**
     * 构造函数
     *
     */
    public function __construct()
    {
        
    }

    public function setDB($conn)
    {
        $this->conn = $conn;
    }
}
```

使用时：

```php
<?php

use Linnzh\Db\SingleConnection;
use Linnzh\Model\Photo;

$photo = new Photo();
$conn = SingleConnection::getInstance();
$photo->setDB($conn);
```

现在，我们的类已经不依赖于特定的数据库连接了。同时，这种方式也易于测试，我们只需要调用`setDB()`方法即可模拟数据库，在`Photo`类的外部，我们拥有完全控制权，具有更高的灵活性和可用性。

类比小游戏“俄罗斯方块”，一开始使用硬编码数据库连接的类，就像一个具有复杂形状的块，而修改后的类则像一个小小的、独立的方块。二者哪个更易于“消除方块”，显而易见。而这大概也有着“组件化思想”的影子。

> Dependency Injection is where components are given their dependencies through their constructors, methods, or directly into fields.

---

问题还没有结束。

当我们开始给`Photo`类添加其他设置功能时，使用者在使用这个类时将十分不便！假设我们的`Photo`类现在是这样：

```php
<?php

namespace Linnzh\Model;

class Photo
{
    protected $conn;# 数据库连接实例

    public function __construct()
    {
        // init code
    }

    public function setDB($conn)
    {
        $this->conn = $conn;
    }

    public function setConfig($config)
    {
        // code
    }

    public function setResponse($response)
    {
        // code
    }
}
```

使用者必须知道所有的依赖项，并记住相应的设置。很糟糕，根据依赖项注入模式的规则，使用`Photo`类必须要做以下事情：

```php
<?php

use Linnzh\Db\SingleConnection;
use Linnzh\Model\Photo;

$photo = new Photo();
$conn = SingleConnection::getInstance();
$photo->setDB($conn);
$photo->setConfig($config);
$photo->setResponse($response);
```

类是模块化了，但使用却变得复杂且混乱。明明之前只需要`new`一个对象即可。

### 解决方案

解决方案是**添加一个容器类来为我们解决初始化的工作**。

如果您了解过“**控制反转**”（Inversion of Control，IoC），那您可能知道这是为什么。

> 译者注：打个比方，A 类的行为完全依赖于 B 类，还是之前的例子，你要起床，依赖时钟，敷面膜，依赖时钟，烤个煎饼也依赖时钟。某一天你外出了，在酒店想敷面膜，时钟不在，就完全做不了其他事情（当然这里只是打比方），这就导致了深度依赖，也就是**过度耦合**。而**控制反转**要做的就是，把主导权移交到自己手里。做法就是上面的**依赖注入**：将依赖倒转为一个普普通通的可选项（就像上面的数据库连接）。在刚才的场景里，你可以将“时钟”设为可选项，注入方法，你想敷面膜就敷，想用时钟来计时或者手机来计时都可以，一切都由你自己做主。这就是**控制反转**。

这个容器类将存储项目中的所有依赖项，并进行相应的“注册”。

我们可以显式地定义一个方法，例如`newPhoto()`：

```php
<?php

namespace Linnzh;
use Linnzh\Model\Photo; 

class Container
{
    protected $db;

    public static function newPhoto()
    {
        $photo = new Photo();
        $photo->setDB(static::$db);
        // $photo->setConfig();
        // $photo->setResponse();

        return $photo;
    }
}

$photo = Container::newPhoto();
```

现在所有的依赖项都在`newPhoto()`方法中配置好了，而不再需要手动配置，我们只需要调用这个方法即可得到一个完全配置的`Photo`类实例。

> 事实上上面代码是错误的，容器类`Container`并没有静态变量`$db`。并且这种方式也不够灵活，需要不断添加新的类实例定义或者修改。


还有一种方式是**编写一个通用的“注册表容器”**，而不是为每个类编写创建新实例的方法。如下所示：

```php
<?php

namespace Linnzh;

class Container
{
    protected static $registry = [];

    /**
     * 注册依赖项
     *
     * @param string $name       依赖名称
     * @param Closure $resolve   创建该依赖项实例的闭包函数
     * @return void
     */
    public static function register(string $name, Closure $resolve)
    {
        self::$registry[$name] = $resolve;
    }

    /**
     * 创建实例
     *
     * @param string $name       依赖项名称，也是注册表 $registry 的键名
     * @return mixed
     */
    public static function resolve(string $name)
    {
        if(static::registered($name)) {
            $name = static::$registry[$name];
            return $name();
        }
        throw new Exception('该名称并未被注册');
    }

    /**
     * 确认该依赖是否已注册
     *
     * @param string $name      依赖项名称，也是注册表 $registry 的键名
     * @return bool             返回是否存在该依赖
     */
    public static function registered(string $name)
    {
        return array_key_exists($name, static::$registry);
    }
}
```

不要被这段代码吓到。这仅仅只是一个向数组（注册表`$register`）中添加 ID 及其关联的解析器的方法。这个解析器只是一个 lambda （注：我觉得更像一个闭包），用来创建实例，并对这个类（要创建的实例的类）设置一切必要的依赖项。

现在我们可以用类似下面的代码来创建`Photo`实例：

```php
<?php

// ...

// 将数据库连接实例加进注册表
Container::register('db', function() {
    return SingleConnection::getInstance();
});

// 将 Photo 类添加进注册表
Container::register('photo', function() {
    $photo = new Photo();
    $photo->setDB(Container::resolve('db'));
    $photo->setConfig();
    $photo->setResponse();
    return $photo;
});
// 获取一个 Photo 类实例
$photo = Container::resolve('photo');
```

这是不是已经开始像许多框架的入口文件了？

使用容器类注册类实例，使得我们不再需要手动实例化类，而是在容器中注册该类，然后请求这个特定的实例。**这个过程就像建立一个数组然后往数组中添加元素，最后根据键名取出这个元素一样简单**。当我们从类中将硬编码的依赖从中剥离，复杂性也随之降低了。

```php
// Before
$photo = new Photo();
 
// After
$photo = Container::resolve('photo');
```

代码量差不多但类变得更加灵活，也便于测试了✌。

<!-- Mark -->
在实际使用中，可能会需要扩展该类，以允许创建单例。（注：这句话没有很理解）


### 进阶：使用魔术方法

如果想进一步减少`Container`容器类的代码量，也避免一些异常，可以使用 PHP 的魔术方法`__set()`和`__get()`。它们将在用户调用不存在的方法时触发。

利用这个特性，我们可以制作一个“自动注册”的效果：

```php
<?php

namespace Linnzh;

use Closure;

class Container
{
    protected $registry = [];

    /**
     * 注册项并不存在时自动触发：向注册表中添加
     *
     * @param string $name       依赖项名称，也是注册表 $registry 的键名
     * @param Closure $resolver  创建该依赖项实例的闭包函数
     */
    public function __set(string $name, Closure $resolver)
    {
        $this->registry[$name] = $resolver;
    }

    /**
     * 读取不可访问或不存在的属性时自动触发：调用创建实例的方法
     *
     * @param string $name
     * @return void
     */
    public function __get(string $name)
    {
        return $this->registry[$name]();
    }
}
```

可以看到上面的代码中，“当注册项不存在时访问注册项”将得到`null`结果（因为不存在），我们给它加上一个反馈：

```php
<?php

namespace Linnzh;

use Closure;
use Exception;

class Container
{
    protected $registry = [];

    /**
     * 注册项并不存在时自动触发：向注册表中添加
     *
     * @param string $name       依赖项名称，也是注册表 $registry 的键名
     * @param Closure $resolver  创建该依赖项实例的闭包函数
     */
    public function __set(string $name, Closure $resolver)
    {
        $this->registry[$name] = $resolver;
    }

    /**
     * 读取不可访问或不存在的属性时自动触发：调用创建实例的方法
     *
     * @param string $name
     * @return void
     */
    public function __get(string $name)
    {
        if(!empty($this->registry[$name]))
        {
            return $this->registry[$name]();
        }
        throw new Exception('该名称并未被注册');
    }
}
```

现在我们可以使用它：

```php
<?php

// ...

// 创建一个容器实例
$container = new Container();
// 向容器内注册 db 实例
$container->db = function() {
    $conn = SingleConnection::getInstance();
    return $conn;
};

// 向容器内注册 photo 实例
$container->photo = function() use($container){
    $photo = new Photo();
    $photo->setDB($container->db);
    $photo->setConfig();
    $photo->setResponse();
    return $photo;
};
// 使用 photo 实例
$photo = $container->photo;
```

大功告成！

## 一个简单的依赖注入实例

很小的一个案例。我们设定了一个数据库连接类（`SingleConnection.php`，位于`path/src/Db/SingleConnection.php`）和一个模型（`Movie.php`，位于`path/src/Model/Movie.php`），代码分别如下：

```php
<?php

declare(strict_types=1);

namespace Linnzh\Db;

use PDO;
use PDOException;

class SingleConnection
{
    const ERROR_UNABLE = '<br>数据库连接失败！<br>';
    private static $instance = NULL;
    private $conn;

    private function __construct(array $dbconfig)
    {
        try {
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
            $this->conn = new PDO($dsn, $dbconfig['username'], $dbconfig['passwd'], $options);
        } catch (PDOException $e) {
            error_log($e->getMessage());
            echo self::ERROR_UNABLE;
            return FALSE;
        }
        return $this->conn;
    }

    private function __clone()
    {
        return $this;
    }

    /**
     * 获取数据库连接实例
     *
     * @param  array $config  以数组形式传入数据库连接参数
     * @return PDO
     */
    static public function getInstance(array $config): PDO
    {
        if (!self::$instance) {
            self::$instance = new self($config);
        }
        return self::$instance->conn;
    }
}

```

**注意修改`getInstance()`方法中的数据库配置参数**。

这里我使用了一个`Movie`表，SQL 语句如下：

```sql
DROP TABLE IF EXISTS `movie`;
CREATE TABLE `movie` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '豆瓣高分电影',
  `name` varchar(60) COLLATE utf8mb4_bin NOT NULL COMMENT '电影名称',
  `rate` float(2,1) unsigned NOT NULL DEFAULT '0.0' COMMENT '豆瓣评分',
  `url` varchar(150) COLLATE utf8mb4_bin NOT NULL DEFAULT '' COMMENT '电影豆瓣网址',
  `cover` varchar(255) COLLATE utf8mb4_bin NOT NULL DEFAULT '' COMMENT '封面图片',
  `cover_x` int(4) unsigned NOT NULL DEFAULT '0' COMMENT '封面横向尺寸',
  `cover_y` int(4) unsigned NOT NULL DEFAULT '0' COMMENT '封面纵向尺寸',
  PRIMARY KEY (`id`),
  KEY `i_name` (`name`),
  KEY `i_rate` (`rate`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='电影';
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (1, '哪吒之魔童降世', 8.5, 'https://movie.douban.com/subject/26794435/', 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2563780504.jpg', 5594, 8268);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (2, '我不是药神', 9.0, 'https://movie.douban.com/subject/26752088/', 'https://img9.doubanio.com/view/photo/s_ratio_poster/public/p2561305376.jpg', 2810, 3937);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (3, '续命之徒：绝命毒师电影', 8.3, 'https://movie.douban.com/subject/30372377/', 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2569548689.jpg', 1500, 2222);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (4, '肖申克的救赎', 9.7, 'https://movie.douban.com/subject/1292052/', 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p480747492.jpg', 2000, 2963);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (5, '这个杀手不太冷', 9.4, 'https://movie.douban.com/subject/1295644/', 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p511118051.jpg', 658, 980);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (6, '绿皮书', 8.9, 'https://movie.douban.com/subject/27060077/', 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2549177902.jpg', 2000, 3167);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (7, '摔跤吧！爸爸', 9.0, 'https://movie.douban.com/subject/26387939/', 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2457983084.jpg', 4500, 6300);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (8, '千与千寻', 9.3, 'https://movie.douban.com/subject/1291561/', 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2557573348.jpg', 1080, 1560);
INSERT INTO `movie`(`id`, `name`, `rate`, `url`, `cover`, `cover_x`, `cover_y`) VALUES (9, '疯狂动物城', 9.2, 'https://movie.douban.com/subject/25662329/', 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2315672647.jpg', 1418, 2005);
```

Movie 模型类我们只简单设定一个输出所有电影信息的方法：

```php
<?php

namespace Linnzh\Model;

use PDO;

class Movie
{
    public $name;
    public $age = 18;

    public function setMovie(string $name)
    {
        $this->name = $name;
    }

    public function getMovies(PDO $conn)
    {
        $sql = 'SELECT * FROM `movie`';
        $stmt = $conn->prepare($sql);
        $stmt->execute();
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}
```

> 注：如果想直接调用，可以删掉命名空间的定义。

然后我们使用容器类`Container.php`（位于`path/src/Container.php`），代码如下：

```php
<?php

namespace Linnzh;

use Closure;

class Container
{
    protected static $registry = [];

    /**
     * 注册项并不存在时自动触发：向注册表中添加
     *
     * @param string  $name       依赖项名称，也是注册表 $registry 的键名
     * @param Closure $resolver   创建该依赖项实例的匿名函数
     */
    public function __set(string $name, Closure $resolver)
    {
        static::$registry[$name] = $resolver;
    }

    /**
     * 读取不可访问或不存在的属性时自动触发：调用创建实例的方法
     *
     * @param string $name
     * @return void
     */
    public function __get(string $name)
    {
        return static::$registry[$name]();
    }
}
```

现在我们写一个测试文件`test.php`：

```php
<?php

require './vendor/autoload.php';

use Linnzh\Container;
use Linnzh\Db\SingleConnection;
use Linnzh\Model\Movie;

// 创建一个容器实例
$container = new Container();

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


// 向容器内注册 db 实例
$container->db = function() use($dbconfig) {
    $conn = SingleConnection::getInstance($dbconfig);
    return $conn;
};


// 测试 数据库连接实例
$sql = 'SELECT * FROM `movie`';
$stmt = $container->db->prepare($sql);
$stmt->execute();
// var_dump($stmt->fetchAll(PDO::FETCH_ASSOC));


// 向容器内注册 Movie 模型
$container->movie = function() {
    return new Movie();
};
$movies = $container->movie->getMovies($container->db);
var_dump($movies);
```

（确保数据库正确连接，并且有对应数据）可以看到正确输出！
