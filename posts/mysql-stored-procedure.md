# MySQL存储过程（Stored Procedure）

## Reference

- [CREATE PROCEDURE](https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html)
- [Using Stored Routines](https://dev.mysql.com/doc/refman/8.0/en/stored-routines.html)
- [Mysql之存储过程与存储函数](https://juejin.im/post/5cdc20ede51d453ce71f6113)

------------------------------------------------------------------------------------------------------------------------

## 前情

在写PHP的一个方法`PDOStatement::bindParam()`文档时，接触到**存储过程**的概念，以前没听说也没用过。后来查了查文档，做个记录。

## MySQL存储过程（Stored Procedure）

一种在数据库中定义存储复杂过程、以便外部程序调用的程序。也可以看作是一个可提供传参和返回参数的、可完成特定功能的 SQL 语句集。

#### 优势

- 存储过程相当于**封装**，可隐藏复杂的逻辑，外部可简单调用
- 存储过程可传值，也可返回值
- 存储过程是一个子程序，不同于函数，不能被 SELECT 指令运行
- 可避免重复的数据库代码，在因更改架构而进行更新、调整查询性能或为日志记录、安全性等添加新的数据库操作时节省精力和时间

#### 缺点

- 过于定制化，各数据库系统间不好通用
- 性能受限于数据库系统

## 语法

```sql
CREATE
    [DEFINER = user]
    PROCEDURE db_name.sp_name ([proc_parameter[,...]])
    [characteristic ...] routine_body
-- 创建存储过程：
-- 定义者，默认为当前用户
-- 如果要与数据库显式关联，则需要显式指定数据库名 db_name；sp_name 为存储过程名


proc_parameter:
    [ IN | OUT | INOUT ] param_name type
-- 存储过程参数：
-- IN 参数表示：传值
-- OUT 参数表示：回值，初始值为 NULL
-- INOUT 参数表示：调用者传参并赋予初始值，在例程中修改，返回修改后的值


type:
    -- 任意有效的 MySQL 数据类型


characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
-- 特征：
-- COMMENT：描述存储过程
-- LANGUAGE：编写SQL例程的语言，仅限 SQL
-- DETERMINISTIC：如果例程对于相同的输入参数始终产生相同的结果，则将其视为“确定性”，否则将其视为“不确定性”。默认为不确定
-- CONTAINS SQL：声明例程不包含读取或写入数据的语句
-- NO SQL：声明例程不包含SQL语句
-- READS SQL DATA：声明例程只读取数据不写入数据
-- MODIFIES SQL DATA：声明例程可能包含写入数据
-- SQL SECURITY：例程使用 DEFINER （定义者）的权限还是 INVOKER （调用者）的权限来执行


routine_body:-- 例程，即存储过程体
    -- 有效的SQL例程语句，由 BEGIN ... END 声明开始与结束
```

## 示例

假设我们有一个`movie`表和一个`comment`表，表结构分别如下：

```sql
DROP TABLE IF EXISTS `movie`;
-- 电影
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

INSERT INTO `movie` VALUES (1, '哪吒之魔童降世', 8.5, 'https://movie.douban.com/subject/26794435/', 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2563780504.jpg', 5594, 8268);
INSERT INTO `movie` VALUES (2, '我不是药神', 9.0, 'https://movie.douban.com/subject/26752088/', 'https://img9.doubanio.com/view/photo/s_ratio_poster/public/p2561305376.jpg', 2810, 3937);
INSERT INTO `movie` VALUES (3, '续命之徒：绝命毒师电影', 8.3, 'https://movie.douban.com/subject/30372377/', 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p2569548689.jpg', 1500, 2222);


DROP TABLE IF EXISTS `comment`;
-- 评论
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

INSERT INTO `comment` VALUES (3, 1, '谢谢你们的鱼', 10591, 5, '2019-07-17 21:22:38', '牛逼！牛逼！牛逼！4年前翻着白眼看完了《大圣归来》，4年后热泪盈眶地看完这部《哪吒之魔童降世》。没想到2019不仅仅是中国科幻大片元年，本片更是和《白蛇：缘起》一块儿创造了国产动漫作品前所未有的高潮。扎实的故事，生动的角色，饱满的情绪，炸裂的场面，这一切都构成了110分钟云霄飞车式的、畅快淋漓的观影体验，表面上是哪吒的传奇，骨子里面是《绝代双骄》+《悟空传》，毁天灭地的逆天气势更是燃破天际，好一曲震撼人心的“冰与火之歌”！都给老子去看！');
INSERT INTO `comment` VALUES (4, 1, '影志', 6874, 5, '2019-07-14 21:19:36', '“不成，功变成仁”、《神仙的自我修养》、“申公公”、内服外敷效果都一样的解药…前面快笑死；指点江山笔、英气的敖丙、猪鼻子倒带器…中间各种神来之笔；“人心的偏见是座大山，你是谁只有你自己才说了算”、父母面前的生离死别…结尾又被感动得一把老泪。老妈的性格更立体，老爸李靖对哪吒说你的真实身份其实是…时，生怕他说出个“蛋”，这是这部毫不正经却又硬气热血至死的片子能干出来的事。集丧逼和燃于一身，在哪吒面前，没人帅的起来！锁定今年最佳国产动画。');
INSERT INTO `comment` VALUES (5, 1, '阿珂', 6388, 4, '2019-07-19 11:38:43', '这个夏天还有比“藕饼”cp更火的吗？');
INSERT INTO `comment` VALUES (6, 1, '徐若风', 3725, 4, '2019-07-18 21:23:14', '客观上的确是目前国产三维动画中的最佳。和年初的《白蛇》一道，富足想象力之下的经典魔改，在人物、情节和视听呈现上都冲着一股劲。不夸张地说，这部作品在今年全球范围动画中比，都可以排到最前列。但还是有些遗憾，双生的概念没有做得更加复杂深刻、部分低俗笑料虽有趣但跳线。可能是我期待太高了。');
INSERT INTO `comment` VALUES (7, 1, '明安', 3335, 4, '2019-07-28 22:05:56', '“所有龙把鳞片扣下来的时候，我真的感觉敖丙压力太大了。”“我们全村可就指望你一个人去考大学了啊，一定要上清华啊。”“就算把其他报清华的人都杀掉你也一定要考上啊！”');
INSERT INTO `comment` VALUES (8, 1, '使君子', 4898, 5, '2019-07-19 11:37:10', '休说苍天不由人，我命由我不由天。新的故事，改编的很不错啊，就是有点短。友情提醒：观影记得带纸巾');
INSERT INTO `comment` VALUES (9, 1, '有心打扰', 1539, 5, '2019-07-14 20:33:33', '神话故事新编，以一种不令人悲伤的方式来讲述悲伤的哪吒童年。丰富多元充满想象力画面；包含幽默、热血、感人和人生观点的剧情；还有安排得当的叙事节奏，结尾处高潮更是一波接一波，让观众又是激动万分，又是感动落泪，实在令人佩服，这就是一部优秀的动画电影。');
INSERT INTO `comment` VALUES (416, 2, 'Peter Cat', 1256, 3, '2018-07-01 18:38:59', '对《药神》所有褒奖都建立在中国电影市场的贫乏和落后之上，纯从视听剧作的技艺层面，《药神》只是一个中规中矩没犯错的工业标准产品，甚至在导演层面是平庸的，连一个令人印象深刻的调度或者美学设计都找不到，如何可能有欲望再看一遍呢？但它又给中国工业指向了一个正确方向...');
INSERT INTO `comment` VALUES (417, 2, '成知默', 236, 4, '2018-07-06 21:33:53', '我认识一位姑娘，曾只身赴印度买仿药；也听闻不少病人，因药价太高而放弃治疗，并很快去世。本片短板很明显，甚至粗糙，但感谢让更多人关注到这个群体。虽有个光明的结尾，但不应一味强化药厂和患者之间的矛盾，尤其现实是零关税下药价未降，药进医保却因限药而买不到药，但愿一切都不只是粉饰太平。');
INSERT INTO `comment` VALUES (418, 2, '末摘花', 726, 5, '2018-07-04 13:16:34', '意外发现王传君的演技真的很棒');
INSERT INTO `comment` VALUES (419, 2, '九只苍蝇撞墙', 1239, 2, '2018-07-01 16:54:05', '完全没有讽刺的意思，我觉得做为一段儿带上海腔的催泪快板儿书，挺煽的。但是电影意识，没找见。不过如今去影院看电影，大家都不在乎后者有还是没有了。');
INSERT INTO `comment` VALUES (420, 2, '中立的手指', 227, 4, '2018-07-05 20:06:21', '怎么过的审？甚至我公安机关的大Boss在片中都是一个略负面的角色。如果此片叫好又叫座，各路资本一拥而上带火了社会现实电影，广电总局会不会再举屠刀啊');
INSERT INTO `comment` VALUES (421, 2, '西楼尘', 961, 3, '2018-07-04 17:09:27', '病是一种罪，上帝也不能赦免；穷是一种病，药神也无法医治。思慧把脱掉的衣服一件件穿上，黄毛把留长的头发一寸寸剪下，程勇把缺口补上，受益把口罩摘下。法和情谁上谁下，命和钱谁上谁下，灾厄的红药水抹到谁的头顶，侥幸的救世主降到何方人间。许多人终其一生都在寻一种药，没药的世界我们能不能活？');
INSERT INTO `comment` VALUES (422, 2, '南悠一', 811, 2, '2018-07-04 12:53:28', '当看见草坪上奔跑的孩子，你流下了两行“前后紧密相连”的热泪：第一行是说，看见了孩子在草地上奔跑，多好啊；第二行是说，和所有的人类在一起，被草地上奔跑的孩子们所感动，多好啊。（我复制的，没错，这部电影太媚俗了，因为看完你只会觉得自己一定要多挣钱。）');
INSERT INTO `comment` VALUES (603, 3, '[未注销]', 12, 4, '2019-10-11 22:49:54', '该来的都有了，不多不少刚刚好。');
INSERT INTO `comment` VALUES (604, 3, '梅山君', 47, 5, '2019-10-11 21:10:38', '我想，以后想起《绝命毒师》，我的内心肯定有一块地方像阿拉斯加那么安静。');
INSERT INTO `comment` VALUES (605, 3, 'Andrew', 31, 5, '2019-10-11 15:47:59', '情怀打满分');
INSERT INTO `comment` VALUES (606, 3, '非魚', 12, 4, '2019-10-12 16:52:10', '老白的“回归”让本片终于有了灵魂，淡淡的落幕，再也不见~');
INSERT INTO `comment` VALUES (607, 3, '殇潮|Enigma', 30, 5, '2019-10-11 22:01:01', 'So....保持好身材的唯一方式就是英年早逝了？');
```

然后我们想要实现一个“**按电影名获取其所有短评**”的查询过程。当然我们可以用一个 SELECT 语句直接查找，如下：

```sql
SELECT * FROM `comment` WHERE `mid` = 
    (SELECT `id` FROM `movie` WHERE `name` = '哪吒之魔童降世')
```

这是只嵌套了一层的 SELECT 查询，如果嵌套多层查询，编写 SQL 就可能很麻烦，一不小心就会遗漏。但使用**存储过程**来实现的话，大概就是这样：

```sql
-- 临时修改语句分隔符为双斜线
DELIMITER //

-- 创建存储过程
CREATE PROCEDURE get_movie_comments(IN movie_name CHAR(50))
BEGIN
    SELECT `id` INTO @id FROM `movie` WHERE `name` = movie_name;
    SELECT * FROM `comment` WHERE `mid` = @id;
END //

-- 改回语句分隔符为分号
DELIMITER ;
```

看到了吗？大致过程就是，先临时修改 MySQL 的语句分隔符，因为定义存储过程时可能会涉及分号（MySQL最初的语句分隔符），为防止定义提前结束，这里临时修改语句分隔符。
然后`BEGIN...END`中间的运行体，可以看到我们进行了两次查询：
1. 先查询指定电影名称的 id，并将其存储在`@id`变量（使用`INTO`子句）
2. 再查询指定`mid`的所有评论，条件语句中使用了`@id`变量

这样逻辑就清晰许多。

调用存储过程使用`CALL sp_name([param[,...]])`语法。例如上面的存储过程，使用如下：

```sql
CALL get_movie_comments('哪吒之魔童降世');
```

查询结果如图：

![image](/images/mysql-stored-procedure-1.png)
