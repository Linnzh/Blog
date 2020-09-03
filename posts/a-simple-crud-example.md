# 一个简单的 CRUD 示例

## Reference

- [Ajax-Web开发者指南](https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX)
- [PHP-PDO](https://www.php.net/manual/zh/book.pdo.php)
- [Vue.js/PHP/MySQL增删改查CRUD示例代码_恒馨博客](https://towait.com/blog/vuejs-crud-with-php-and-mysql/)

------------------------------------------------------------------------------------------------------------------------

## 前情

总是听说`CRUD`，但一直不清楚是做什么的，就去查了一下，大概的意思是一组常见的数据库操作：**增（create）、查（read）、改（update）删（delete）**，大概是，也有其他的翻译，这里大概了解一下就好。截止到现在，网上好像没有什么很小的示例来阐述`CRUD`这个概念的，然后就去查了一番资料，写了一个真的很小白的、很简单、未使用任何框架的案例。

## 前端准备

由于笔者对前端知识并不熟悉，这里只贴容器（传输/返回数据的容器）代码，在服务器根目录下新建文件`index.html`，代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>CRUD操作</title>
</head>
<body>
    <!-- Read：显示数据 -->
    <button id="read">read</button><br>

    <!-- 引入JavaScript文件 -->
    <script src="./index.js"></script>
</body>
</html>
```

这段代码只有一个按钮，设定为**点击即显示所有的用户**（辣鸡笔者想不出什么合适的示例，先将就着看）。
由于不喜欢在HTML中嵌套PHP代码，故这里使用JavaScript来传递前后端的数据。在同级目录下新建文件`index.js`，代码如下：

```javascript
'use strict';

let read = document.getElementById('read');
if (read) {
    read.addEventListener('click', function () {
        let server = './crud.php';
        let param = 'action=read';
        get(server, param);
    });
}
function get(server, param = '') {
    let request = new XMLHttpRequest();
    request.open('GET', server);
    request.send(param);
    request.onreadystatechange = function () {
        if (request.readyState === 4 && request.status === 200) {
            let data = JSON.parse(request.responseText);
            console.log(data);
        }
    };
}
```

可以看到，上面的JavaScript代码中定义了一个`get()`函数，该函数创建了一个使用`GET`传递对象的`XMLHttpRequest`对象，也就是常说的**Ajax**，这里不做过多赘述，详情可看：[Ajax](https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX)。

而函数`get()`上方的代码则是获取read按钮元素，并监听(`addEventListener`)其点击事件，一旦用户点击该按钮，则访问同目录下的`crud.php`文件，并传递参数`action=read`。
我们设定判断用户操作的参数为`action`，而获取数据的参数值为`read`，然后交给后端处理。

## 后端处理

前端负责传输数据后，后端就该负责处理数据了。

### 1. 建库建表

既然是关于数据库的操作，那么数据库中必须要有数据。这里笔者用的是`user`表，刚建的测试表，只有三行数据，表数据如下：

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称',
  `pwd` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '加密后的密码',
  `gender` tinyint(1) UNSIGNED NOT NULL DEFAULT 1 COMMENT '性别',
  `profile` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '简介',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = 'user' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES (1, 'ds', 'dsdsds', 1, 'dwwadsa');
INSERT INTO `user` VALUES (2, 'dsa', 'wdqdqd', 0, 'dasdfsaf');
INSERT INTO `user` VALUES (3, 'reg.lynn@qq.com', 'dsdsds', 0, '');

SET FOREIGN_KEY_CHECKS = 1;
```

### 2. 连接数据库

```php
<?php

/**
 * 连接数据库实例
 *
 * @param string $dsn
 * @param array $dbconfig
 * @param array $options
 * @return PDO
 */
function getConnection(string $dsn, array $dbconfig, array $options = []): PDO
{
    try {
        $conn = new PDO($dsn, $dbconfig['username'], $dbconfig['passwd'], $options);
        $conn->query('SET NAMES ' . $dbconfig['charset']);

        return $conn;
    } catch (\PDOException $e) {
        throw new PDOException($e->getMessage());
    } catch (\Exception $e) {
        throw new Exception($e->getMessage());
    }
}

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

$conn = getConnection($dsn, $dbconfig, $options);

```

### 3. 处理数据

前面的JavaScript代码中，我们将`CRUD`要访问的文件设置为同级目录下的`crud.php`（实际开发肯定是要分开的，但这里仅作示例），于是，在同级目录下新建文件`crud.php`，并输入以下代码：

```php
<?php

// ...

$action = 'read';// 设置CRUD操作默认值：read

if (isset($_GET['action'])) $action = $_GET['action'];// 如果前端发来了操作代码，则换成前端需要的操作

switch ($action) {// 根据不同操作使用不同的处理方式：这里使用 switch 结构，也可以使用 if
    case 'create':// 前端操作值为 create 时
        break;
    case 'read':// 前端操作值为 read 时
        $sql = 'SELECT * FROM `user`';
        $stmt = $conn->prepare($sql);
        if ($stmt->execute()) {
            $res['user'] = $stmt->fetchAll(PDO::FETCH_ASSOC);
        } else {
            $res['error'] = true;
            $res['message'] = '获取信息失败！';
        }
        break;
    case 'update':// 前端操作值为 update 时
        break;
    case 'delete':// 前端操作值为 delete 时
        break;
}

header("Content-type:application/json");// 告知浏览器返回的数据为JSON格式
echo json_encode($res, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);// 返回JavaScript可解析的JSON数据
```

上面的代码使用注释解释了其用途，这里说一下`case 'read':`代码块。我们在这里使用`PDO`预处理获取user表全部数据的SQL语句（`PDO::prepare()`），然后执行（`PDO::execute()`），最后获取查询的所有行（`PDOStatement::fetchAll()`）并保存在一个数组中。
为避免查询失败后无返回数据，在执行查询结果时添加了错误处理，并给出提示信息（`else`语句块）。最后返回带有数据或错误信息的JSON数据。

这里是笔者的查询结果截图：

![查询结果](/images/a-simple-crud-example-1.png)

## 剩余部分

余下就是添加 create/update/delete 操作时的代码了，具体可查看`a-simple-crud-example`分支。

## 代码提要

### JavaScript：Ajax

```javascript
function post(server, param = '') {
    let request = new XMLHttpRequest();
    request.open('POST', server);
    request.send(param);
    request.onreadystatechange = function () {
        if (request.readyState === 4 && request.status === 200) {
            let data = JSON.parse(request.response);
            console.log(data);
        }
    };
}
```

解析：
- 创建`XMLHttpRequest`对象
- 使用`POST`或`GET`打开服务器`server`
- 发送数据`param`，例如：`name=Linnzh`，`name=Linnzh&age=23`。如果传输的参数包含特殊符号（比如双引号`"`），可使用`encodeURIComponent()`函数进行处理
- 请求成功时处理返回的数据`request.responseText`（纯文本数据），返回JSON字符串时需要使用`JSON.parse()`函数解析为JSON对象

### JavaScript：生成表单

```javascript
let formdata = new FormData();
let name = document.getElementById('name').value;
let pwd = document.getElementById('pwd').value;
let gender = document.getElementById('gender').value;
formdata.append('username', name);
formdata.append('userpwd', pwd);
formdata.append('usergender', gender);
```

为啥要使用这个方法呢？因为不想在HTML文件中仅创建一个`Button`就必须使用一个`from`，而且不能自定义控制数据的传输方式。

解析：

- 创建表单数据对象：`FormData()`
- 获取表单元素（`input`等）的值
- 将表单值添加至表单数据对象：`formdata.append('username', name);`
- 最后使用合适的传输方法进行传输

### JavaScript：一次性发送两种不同传输方式的数据时的解决方案

在这个例子中，增加、删除和更新操作，都需要通过`POST`发送数据（避免明文发送敏感数据），而数据库操作则只需要通过`GET`发送。如果分成两次发送，则会造成数据缺失，也造成重复请求。
因为`GET`方式通常只用来传输简短的、不重要的信息，故在数据量不小的情况下直接添加在要访问的`URL`中，然后`POST`访问该添加了`GET`参数的URL即可，如下：

```javascript
let server = './server/crud.php?action=delete';// 传递数据时可能有POST也可能有GET数据，只能用一次方法，因而将GET参数直接添加在链接中
post(server, formdata);
```

### PHP：PDO预处理

**始终建议使用PDO预处理语句操作数据库**。

```php
$username = $_POST['username'];
$userpwd = $_POST['userpwd'];
$usergender = (int)$_POST['usergender'];
$sql = 'INSERT INTO `user`(`name`,`pwd`,`gender`) VALUES (:name, :pwd, :gender)';
$stmt = $conn->prepare($sql);
$stmt->bindParam(':name', $username);
$stmt->bindParam(':pwd', $userpwd);
$stmt->bindParam(':gender', $usergender);
if ($stmt->execute()) {
    $res['message'] = '数据插入成功！';
}
```

- 预处理SQL：`PDO::prepare($sql)`
- 绑定参数：如果不需要绑定则省略该步。推荐使用命名绑定，`PDOStatement::bindParam()`
- 执行预处理语句：`PDOStatement::execute()`
- 从查询结果集中获取数据：`PDOStatement::fetchAll(PDO::FETCH_ASSOC)`


------------------------------------------------------------------------------------------------------------------------

简单的CRUD示例差不多就是这样，如果有合适的项目+好看的UI就更好了。有机会补上。