# 使用 Markdown 解析库：Parsedown

## Reference

- [Parsedown](https://github.com/erusev/parsedown)

------------------------------------------------------------------------------------------------------------------------

## 前情

在很早很早之前，看了许多博客搭建教程的时候（然而到现在我也没有一个自己搭建的博客），就好奇大佬们的`markdown`文件是怎么转化为HTML的。但因为那时对博客技术兴致缺缺，并没有深入去想、去实验，只隐约感觉是字符串替换之类的。

现在习惯于使用 Markdown 语法写文档，这阵子也遇上这样的需求，于是上网找了找相关的库。对比了几家，决定使用[Parsedown](https://github.com/erusev/parsedown)。这个库的优点在于只有一个文件，并且测试检查之类的做得非常棒（让我这个从没接触过测试的小白有一点点想学测试），当然这是题外话。

现在来看看这个库的基本使用。

## 开始

首先，可以使用包管理工具`cmposer`在项目中添加这个包（截至2019年11月5号，这个库的composer版本为`1.7.3`，在其源代码可看到）：

```bash
composer require erusev/parsedown
```

或者不想通过 composer ，直接`git clone https://github.com/erusev/parsedown.git`这个仓库（仓库版本比包更新，并含有丰富、规范的测试文件）放在项目目录也可以。

或者更功利一点，只获取主要的文件`Parsedown.php`存放在项目目录。这也是我很喜欢这个库的原因——**只有一个文件，也没有其他更多的依赖。**

### 使用

假设我们已经使用 composer 引入这个库了，并在`/markdown`文件夹有一个用来测试的 markdown 文件`demo.md`，内容如下（该内容截取自笔者的另一篇文章：[一个简单的CRUD示例
](/posts/a-simple-crud-example.md)）：

````markdown
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
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = 'user' ROW_FORMAT = Dynamic;

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
    }
}

$conn = getConnection($dsn, $dbconfig, $options);

```
上面的代码使用注释解释了其用途，这里说一下`case 'read':`代码块。

这里是笔者的查询结果截图：

![查询结果](/images/a-simple-crud-example-1.png)

````

然后开始解析。

新建文件`parse.php`，内容如下：

```php
<?php

require_once __DIR__.'/vendor/autoload.php';

$parse = new Parsedown();
$content = file_get_contents('./markdown/demo.md');
$result = $parse->text($content);
echo $result;
```

运行该文件，可查看结果：

```html
<h2>前端准备</h2>
<p>由于笔者对前端知识并不熟悉，这里只贴容器（传输/返回数据的容器）代码，在服务器根目录下新建文件<code>index.html</code>，代码如下：</p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
    &lt;meta charset="UTF-8"&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
    &lt;meta http-equiv="X-UA-Compatible" content="ie=edge"&gt;
    &lt;title&gt;CRUD操作&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;!-- Read：显示数据 --&gt;
    &lt;button id="read"&gt;read&lt;/button&gt;&lt;br&gt;

    &lt;!-- 引入JavaScript文件 --&gt;
    &lt;script src="./index.js"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;</code></pre>
<p>这段代码只有一个按钮，设定为<strong>点击即显示所有的用户</strong>（辣鸡笔者想不出什么合适的示例，先将就着看）。
由于不喜欢在HTML中嵌套PHP代码，故这里使用JavaScript来传递前后端的数据。在同级目录下新建文件<code>index.js</code>，代码如下：</p>
<pre><code class="language-javascript">'use strict';

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
        if (request.readyState === 4 &amp;&amp; request.status === 200) {
            let data = JSON.parse(request.responseText);
            console.log(data);
        }
    };
}</code></pre>
<p>可以看到，上面的JavaScript代码中定义了一个<code>get()</code>函数，该函数创建了一个使用<code>GET</code>传递对象的<code>XMLHttpRequest</code>对象，也就是常说的<strong>Ajax</strong>，这里不做过
多赘述，详情可看：<a href="https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX">Ajax</a>。</p>
<p>而函数<code>get()</code>上方的代码则是获取read按钮元素，并监听(<code>addEventListener</code>)其点击事件，一旦用户点击该按钮，则访问同目录下的<code>crud.php</code>文件，并传递参数<code>action=read</code>。
我们设定判断用户操作的参数为<code>action</code>，而获取数据的参数值为<code>read</code>，然后交给后端处理。</p>
<h2>后端处理</h2>
<p>前端负责传输数据后，后端就该负责处理数据了。</p>
<h3>1. 建库建表</h3>
<p>既然是关于数据库的操作，那么数据库中必须要有数据。这里笔者用的是<code>user</code>表，刚建的测试表，只有三行数据，表数据如下：</p>
<pre><code class="language-sql">SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(30) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '昵称',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = 'user' ROW_FORMAT = Dynamic;
</code></pre>
<h3>2. 连接数据库</h3>
<pre><code class="language-php">&lt;?php

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
        $conn-&gt;query('SET NAMES ' . $dbconfig['charset']);

        return $conn;
    } catch (\PDOException $e) {
        throw new PDOException($e-&gt;getMessage());
    }
}

$conn = getConnection($dsn, $dbconfig, $options);
</code></pre>
<p>上面的代码使用注释解释了其用途，这里说一下<code>case 'read':</code>代码块。</p>
<p>这里是笔者的查询结果截图：</p>
<p><img src="/images/a-simple-crud-example-1.png" alt="查询结果" /></p>
```

是不是标准的 HTML 片段？`Parsedown`只渲染代码块类，类名定义为`language-xx`，我想大概是因为前端代码高亮JS库`highlight.js`的普遍使用。

用起来超级简单的。
