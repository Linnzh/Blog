# php-curl-class 库的使用（简单爬虫）

## Reference

- [php-curl-class](https://github.com/php-curl-class/php-curl-class)
- [curl: (60) SSL certificate : unable to get local issuer certificate](https://stackoverflow.com/questions/24611640/curl-60-ssl-certificate-unable-to-get-local-issuer-certificate)

------------------------------------------------------------------------------------------------------------------------

## 前情

想利用 PHP 进行爬虫（主要是比较熟悉 PHP 的使用）。然后找了找相关的 CURL 库，发现了[php-curl-class](https://github.com/php-curl-class/php-curl-class)，名字很是简单粗暴，于是尝试了一下，暂作记录。

## 开始使用

### 安装依赖

按照其 README.md 文档所描述的，使用 composer 命令在项目中安装依赖：

```bash
composer require php-curl-class/php-curl-class
```

## 项目

这里以**爬取豆瓣高分电影**为项。地址：[https://movie.douban.com/explore#!type=movie&tag=%E8%B1%86%E7%93%A3%E9%AB%98%E5%88%86&sort=time&page_limit=20&page_start=0](https://movie.douban.com/explore#!type=movie&tag=%E8%B1%86%E7%93%A3%E9%AB%98%E5%88%86&sort=time&page_limit=20&page_start=0)

打开浏览器的`F12`，可以看到这样的数据：

![image](/images/try-library-of-php-curl-class-1.png)

很方便，结果都是JSON格式的数据。省去了正则提取的过程。现在开始入库。

### 数据入库

根据返回的信息，我们创建`movie`表：

```sql
DROP TABLE IF EXISTS `movie`;
CREATE TABLE `movie` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '豆瓣高分电影',
  `name` varchar(60) COLLATE utf8mb4_bin NOT NULL COMMENT '电影名称',
  `rate` float(2,1) unsigned NOT NULL DEFAULT '0.0' COMMENT '豆瓣评分',
  `url` varchar(150) COLLATE utf8mb4_bin NOT NULL DEFAULT '' COMMENT '电影豆瓣网址',
  `cover` varchar(255) COLLATE utf8mb4_bin NOT NULL DEFAULT '' COMMENT '封面图片',
  `cover_x` int(255) unsigned NOT NULL DEFAULT '0' COMMENT '封面横向尺寸',
  `cover_y` int(255) unsigned NOT NULL DEFAULT '0' COMMENT '封面纵向尺寸',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_name` (`name`),
  KEY `i_name` (`name`),
  KEY `i_rate` (`rate`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='豆瓣电影：https://movie.douban.com/explore';
```

### 使用 php-curl-class 库获取电影

新建文件`index.php`，代码如下：

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use Curl\Curl;
use Linnzh\Db\SingleConnection;

/**
 * 抓取豆瓣热门电影
 */

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
$curl = new Curl();
$conn = SingleConnection::getInstance($dbconfig);
$insert = 'INSERT INTO `movie`(`name`,`rate`,`cover_x`,`cover_y`,`url`,`cover`) VALUES (:name,:rate,:cover_x,:cover_y,:url,:cover)';
$stmt = $conn->prepare($insert);
for ($start = 0; $start < 20000; $start += 20) {
    $url = "https://movie.douban.com/j/search_subjects?type=movie&tag=%E8%B1%86%E7%93%A3%E9%AB%98%E5%88%86&sort=time&page_limit=20&page_start=$start";
    $curl->get($url);
    $curl->setOpt(CURLOPT_SSL_VERIFYPEER, false);

    if(empty($curl->getResponse())) {
        continue;
    }

    foreach ($curl->getResponse()->subjects as $key => $value) {
        // var_dump($value->title);exit;
        try {
            $stmt->bindParam(':name', $value->title);
            $stmt->bindParam(':rate', $value->rate);
            $stmt->bindParam(':cover_x', $value->cover_x);
            $stmt->bindParam(':cover_y', $value->cover_y);
            $stmt->bindParam(':url', $value->url);
            $stmt->bindParam(':cover', $value->cover);
            $stmt->execute();
        } catch (PDOException $e) {
            if ($e->getCode() == 23000) {
                continue;
            }
        } catch (Exception $e) {
            throw new Exception($e->getMessage());
        }
    }
}
$curl->close();

```

这段爬虫代码仅作示例，还有很多缺陷，例如没有完善的错误处理、没有对其他情况进行分析等。

## 爬取短评

转到某部电影的**全部短评**页面，例如：[https://movie.douban.com/subject/26794435/comments?status=P](https://movie.douban.com/subject/26794435/comments?status=P)，依旧是`F12`键查看其返回数据，可以发现第一页只返回了一个HTML文档，这看起来有点麻烦，于是点开[下一页](https://movie.douban.com/subject/26794435/comments?start=20&limit=20&sort=new_score&status=P)，这时候的服务器返回的是一个 JSON 数组，如图：

![image](/images/try-library-of-php-curl-class-2.png)

我们把键名为`html`的键值复制到一个名为`demo.html`的文件，如下：

```html
<div class="comment-item" data-cid="1860359970">


<div class="avatar">
    <a title="⎝丁凯乐⎠" href="https://www.douban.com/people/52623673/">
        <img src="https://img3.doubanio.com/icon/u52623673-10.jpg" class="" />
    </a>
</div>
<div class="comment">
<h3>
    <span class="comment-vote">
        <span class="votes">30080</span>
        <input value="1860359970" type="hidden"/>
        <a href="javascript:;" class="j a_show_login" onclick="">有用</a>
    </span>
    <span class="comment-info">
        <a href="https://www.douban.com/people/52623673/" class="">⎝丁凯乐⎠</a>
            <span>看过</span>
            <span class="allstar50 rating" title="力荐"></span>
        <span class="comment-time " title="2019-07-16 13:43:25">
            2019-07-16
        </span>
    </span>
</h3>
<p class="">

        <span class="short">实名反对最赞说烂片的评论，这是人类无法逃脱的真香定律！
看完觉得不值票钱可以来快乐星球砍我！</span>
</p>
</div>

</div>

<!-- 省略一部分 -->

<div class="comments-footer-tips">
* 影片上映之前的、与影片无关的或包含人身攻击等内容的短评都将有可能被折叠，且评分不计入豆瓣评分。<br/>
* 短评的排序是将豆瓣成员的投票加权平均计算后的结果，通过算法的调校，更好地反映短评内容的价值。<br/>
</div>


<div id="paginator" class="center">
        <span class="first"><< 首页</span>
        <span class="prev">< 前页</span>
        <a href="?start=20&amp;limit=20&amp;sort=new_score&amp;status=P&amp;percent_type=" data-page="" class="next">后页 ></a>
</div>
```

### 分析文本特征

然后来分析这段文本的特征，从而使用正则表达式提取我们需要的信息。

怎么分析呢？就是找特征点。反正我找完后正则表达式就是这个：

```
#class="avatar">\s*<a title="([^"]*?)".*?class="votes">(\d+)<.*?class="allstar(\d+)\s.*?class="comment-time\s*"\s*title="([^"]*?)".*?class="short">([^<]*?)</span>#is
```

其中以符号`#`作为包围符号，最后的`is`分别表示：
- `i`：表示忽略大小写
- `s`：表示将空白符`\r`、`\n`和`\t`等一律按照空格处理


### 数据入库

分析完文本特征后，我们也可以提取相关信息存入数据库，表字段如下：

```sql
DROP TABLE IF EXISTS `comment`;
CREATE TABLE `comment` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '评论ID',
  `mid` int(10) unsigned NOT NULL COMMENT '电影ID',
  `username` varchar(40) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '用户名',
  `votes` int(5) DEFAULT NULL COMMENT '该评论投票数',
  `score` int(2) NOT NULL COMMENT '打分',
  `time` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL COMMENT '发布评论的时间',
  `content` varchar(255) COLLATE utf8mb4_bin NOT NULL COMMENT '评论内容',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_mid_user` (`mid`,`username`),
  KEY `i_score` (`score`),
  KEY `i_content` (`content`),
  KEY `i_time` (`time`),
  KEY `i_vote` (`votes`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='豆瓣电影-评论表';
```

很好，现在我们可以写抓取短评的代码了。


### 使用 php-curl-class 库抓取短评

新建一个文件`comments.php`，抓取电影的详情url从`movie`表中获取。代码如下：

```php
<?php

require '../vendor/autoload.php';

use Curl\Curl;
use Linnzh\DB\Connection;

/**
 * 抓取豆瓣电影短评
 */

$sql = 'SELECT `url`,`id` FROM `movie` ORDER BY `id`';
$conn = Connection::getInstance();
$stmt = $conn->prepare($sql);
$stmt->execute();

$regex = '#class="avatar">\s*<a title="([^"]*?)".*?class="votes">(\d+)<.*?class="allstar(\d+)\s.*?class="comment-time\s*"\s*title="([^"]*?)".*?class="short">([^<]*?)</span>#is';
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    for ($start = 0; $start < 200; $start += 20) {
        $url = $row['url'] . "comments?start=$start&limit=20&sort=new_score&status=P&comments_only=1";
        $curl = new Curl();
        $curl->get($url);
        $curl->setOpt(CURLOPT_SSL_VERIFYPEER, false);
        $curl->setUserAgent('');
        // var_dump($curl->response);exit;
        $curl->close();
        // var_dump($curl->getResponse()->html);
        preg_match_all($regex, $curl->getResponse()->html, $matches);
        // var_dump($matches);exit;
        foreach ($matches[0] as $key => $value) {
            $data = ['mid' => $row['id'], 'username' => $matches[1][$key], 'votes' => $matches[2][$key], 'score' => (int) $matches[3][$key] / 10, 'time' => $matches[4][$key], 'content' => $matches[5][$key]];
            $cols = '`' . implode('`,`', array_keys($data)) . '`';
            $values = "'" . implode("','", array_values($data)) . "'";
            $insert = "INSERT INTO `comment`($cols) VALUES ($values)";
            $conn->prepare($insert)->execute();
        }
        // exit;
    }
}
```

由于部分电影短评数量过多，这里可能抓取不完全。

### 注意

笔者在第一次使用[php-curl-class](https://github.com/php-curl-class/php-curl-class)库时，出现错误提示`error code: 60 error message: SSL certificate problem: unable to get local issuer certificate`（无法找到本地的 SSL 证书）。解决方法请参考：[curl: (60) SSL certificate : unable to get local issuer certificate](https://stackoverflow.com/questions/24611640/curl-60-ssl-certificate-unable-to-get-local-issuer-certificate)。
