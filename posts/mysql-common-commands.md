# MySQL 常用命令

## Reference

- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/tutorial.html)

------------------------------------------------------------------------------------------------------------------------

## 用户

### 创建用户

```sql
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'mysql';

CREATE USER 'user'@'%' IDENTIFIED BY 'mysql';
```

### 用户授权

```sql
GRANT ALL ON `test`.* TO 'admin'@'localhost' WITH GRANT OPTION;

GRANT SELECT, UPDATE, INSERT, DELETE ON `test`.* TO 'user'@'%' WITH GRANT OPTION;
```

### 取消用户权限

```sql
REVOKE DELETE ON `test`.* FROM 'user'@'%';
```

### 删除用户

```sql
DROP USER 'user'@'%';
```

------------------------------------------------------------------------------------------------------------------------

## 数据库

### 登录数据库

```sql
mysql -u admin -p mysql
```

### 创建数据库

```sql
CREATE DATABASE `test` CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
```

### 查看该用户下所有的数据库

```sql
SHOW DATABASES;
```

### 切换数据库

```sql
USE test;
```

### 删库

```sql
DROP DATABASE IF EXISTS `db`;
```

------------------------------------------------------------------------------------------------------------------------

## 表

### 建表

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `test`.`user`  (
  `id` int(0) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(30) NOT NULL COMMENT '昵称',
  `pwd` varchar(32) NOT NULL COMMENT '加密后的密码',
  `gender` tinyint(1) NOT NULL COMMENT '性别',
  `profile` varchar(255) NOT NULL COMMENT '简介',
  PRIMARY KEY (`id`)
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = 'user';
```

### 删表

```sql
DROP TABLE `user`;
```

### 清空表

```sql
-- 无法回退，删除表数据和结构并重建自动增长为1的新表
TRUNCATE `tableName`;
```

```sql
-- 在事务中可回退，较慢，自动增长延续之前的ID值
DELETE `tableName`;
```

### 插入行

```sql
INSERT INTO `test`.`user`(`id`, `name`, `pwd`, `gender`, `profile`) 
VALUES (1, 'Linnzh', 'e10adc3949ba59abbe56e057f20f883e', 1, 'Linnzh是一个超级可爱超级聪明的绿孩子！！！');
```

### 删除行

```sql
DELETE FROM `test`.`user` WHERE `id` = 1;
```

### 更新行

```sql
UPDATE `test`.`user` SET `pwd` = 'c33367701511b4f6020ec61ded352059' WHERE `id` = 1;
```

### 创建索引

```sql
ALTER TABLE `test`.`user` 
ADD INDEX `idx_name`(`name`) COMMENT '昵称索引';
```

```sql
CREATE INDEX `idx_name` ON `test`.`user`(`name`);
```

### 删除索引

```sql
ALTER TABLE `test`.`user` 
DROP INDEX `idx_name`;
```

```SQL
DROP INDEX `idx_name` ON `test`.`user`;
```

------------------------------------------------------------------------------------------------------------------------

## SELECT

```sql
SELECT * FROM `test`.`user`;
```

```sql
SELECT `id`, `name` FROM `test`.`user` WHERE `gender` = 1;
```

```sql
SELECT count(*) AS `count` FROM `test`.`user`;
```

------------------------------------------------------------------------------------------------------------------------

## WHERE

```sql
-- 普通 WHERE 条件
SELECT `id`,`name` FROM `people` WHERE `id` > 1000;
```

```sql
-- LENGTH() 函数：查询`name`字段长度能被 3 整除（在 UTF-8 字符集中，汉字占 3 个字节）的记录
SELECT `id`,`name` FROM `people` WHERE LENGTH(`name`) % 3 = 0;
```

```sql
-- REGEXP() 函数：查询`pwd`字段内容符合正则表达式（这里是数字+字母大小写）的记录
SELECT `id`,`name`,`pwd` FROM `user` WHERE `pwd` REGEXP('[0-9A-Za-z]');
```

```sql
-- UNIX_TIMESTAMP() 函数：将一个日期字符串格式化为 unix 时间戳
SELECT * FROM `user` WHERE `regtime` > UNIX_TIMESTAMP('2019-12-10');
```

```sql
-- IN 子句：查询`name`字段是 IN 子句括号内的内容那个之一的记录
SELECT `id`,`name`,`rate` FROM `people` WHERE `name` IN('李大成','张悦');
```

```sql
-- ORDER LIMIT OFFSET：分页查询
SELECT * FROM `user` ORDER BY `id` LIMIT 10 OFFSET 0
```

------------------------------------------------------------------------------------------------------------------------

## ALTER

```sql
-- 添加列
ALTER TABLE `book` ADD COLUMN `addtime` INT(10) UNSIGNED NOT NULL DEFAULT 0 COMMENT '该书籍添加时间';

-- 修改列信息
ALTER TABLE `book` MODIFY COLUMN `update` INT(10) UNSIGNED NOT NULL DEFAULT 0 COMMENT '该书籍最近更新时间';

```

------------------------------------------------------------------------------------------------------------------------
