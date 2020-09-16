# 开始学习 Symfony

## Reference

- [Databases and the Doctrine ORM](https://symfony.com/doc/current/doctrine.html)
- [Testing](https://symfony.com/doc/current/testing.html)
- [How to Test Code that Interacts with the Database](https://symfony.com/doc/current/testing/database.html)

-------------------------------


## 安装

## 关于 .env 文件

在 Laravel 框架中，`.env`属于正式环境的环境文件，不纳入代码管理范畴，而是以`.env.example`的形式提供示例。并且引入`DotEnv`库来解析环境配置文件，使用非常简单。

但 Symfony 不同。



## 使用 Doctrine

#### 1. 引入

```bash

composer require symfony/orm-pack
composer require --dev symfony/maker-bundle

```

#### 2. 配置

```
# .env

APP_ENV=dev
APP_SECRET=

DATABASE_URL=mysql://root:password@127.0.0.1:3306/project_manage?serverVersion=8.0&charset=utf8mb4&collate=utf8mb4_unicode_ci
```

#### 3. 创建数据库

```bash
php bin/console doctrine:database:create -e test
```

#### 4. 创建实体

```bash
php bin/console make:entity
```

其中实体类的部分注解如下：

#### 5. 映射表并创建表迁移文件

```bash
php bin/console make:migration

# 执行迁移
php bin/console doctrine:migrations:migrate
```

更新实体可重复 4~5 步骤。

## 使用测试框架


## 关于服务
