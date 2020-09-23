# 开始学习 Symfony

## Reference

- [Databases and the Doctrine ORM](https://symfony.com/doc/current/doctrine.html)
- [Testing](https://symfony.com/doc/current/testing.html)
- [How to Test Code that Interacts with the Database](https://symfony.com/doc/current/testing/database.html)
- [Working with Associations](https://www.doctrine-project.org/projects/doctrine-orm/en/2.7/reference/working-with-associations.html)

-------------------------------


## 安装

```bash
composer create-project symfony/skeleton my_project_name
```

## 关于 .env 文件

在 Laravel 框架中，`.env`属于正式环境的环境文件，不纳入代码管理范畴，而是以`.env.example`的形式提供示例。并且引入`DotEnv`库来解析环境配置文件，使用非常简单。

但 Symfony 不同。

`.env`是正式环境的环境文件，纳入代码管理范畴。在此基础之上，还有`.env.local`环境文件，表示本地使用的环境配置，不纳入代码管理。
并且提供了另外两种不同环境：`.env.dev`-开发环境、`.env.test`-测试环境。在后面添加`.local`则表明是本地配置。
本地配置优先于其他环境配置。

在 symfony 的控制台命令中，有一个`--env`/`-e`的环境变量选项可配置，默认为`dev`开发环境。


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

# secret 可通过命令 php bin/console secrets:generate-keys 生成
APP_SECRET=

DATABASE_URL=mysql://root:password@127.0.0.1:3306/project_manage?serverVersion=8.0&charset=utf8mb4&collate=utf8mb4_unicode_ci
```

> **注意**：

在环境变量文件中配置的常量，需要在 `config/serivice.yaml` 文件中配置，如下：

```yaml
# config/serivice.yaml
parameters:
    external_host: '%env(resolve:EXTERNAL_HOST)%'
    external_url: '%env(resolve:EXTERNAL_URL)%'
```

添加/修改环境变量后，需执行`composer dump-env dev/prod/test`来更新环境变量。

而在代码中使用环境变量时，需要使用`$this->getParameter('external_host')`形式（这里是在容器中使用，在其他地方使用请参考：<https://symfony.com/doc/current/configuration.html>

#### 3. 创建数据库

```bash
php bin/console doctrine:database:create -e test
```

#### 4. 创建实体

```bash
php bin/console make:entity
```

其中实体类的部分注解如下：

```php
// 默认值在 options 中设置，Doctrine 并不根据实体类属性默认值添加默认值
@ORM\Column(type="string", length=255, options={"default"=""})

@ORM\Column(type="decimal", precision=10, scale=2, options={"default"=0.00})

@ORM\Column(type="integer", options={"default"=0})


    /**
     * 关联-多对一
     * @ORM\ManyToOne(targetEntity=ProjectDetail::class, inversedBy="products")
     * @ORM\JoinColumn(nullable=false)
     */
    private $projectDetail;
```

#### 5. 映射表并创建表迁移文件

```bash
php bin/console make:migration

# 执行迁移
php bin/console doctrine:migrations:migrate
```

如果实体发生变更，例如更改注释、更改列名，可通过`php bin/console make:entity --regenerate`来重新生成实体（根据属性生成对应的`getter()`/`setter()`方法）。
然后继续执行第五步，生成迁移并执行。

## 使用测试框架

1. 普通测试示例

```php
<?php

namespace App\Tests\Repository;

use PDO;
use PDOException;
use PHPUnit\Framework\TestCase;

class ConnectionTest extends TestCase
{
    public function testSomething()
    {
        $dsn = "mysql:host=127.0.0.1;dbname=mysql";
        $user = 'root';
        $passwd = 'password';
        $options = [
            PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4',
        ];
        try {
            $conn = new PDO($dsn, $user, $passwd, $options);
            $stmt = $conn->query('SHOW DATABASES');
            $result = $stmt->fetchAll();
            $this->assertIsArray($result, json_encode($result, JSON_UNESCAPED_UNICODE));
        } catch (PDOException $e) {
            throw new PDOException($e->getMessage());
        }
    }
}
```

2. WEB 请求测试

```php
<?php

namespace App\Tests\Http;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class RequestTest extends WebTestCase
{
    public function testSomething()
    {
        $client = static::createClient();
        // $client->request('GET', 'http://localhost:8080/test/hello');
        $client->request('GET', '/test/hello');

        $this->assertResponseIsSuccessful();
        $this->assertEquals(200, $client->getResponse()->getStatusCode());
        $this->assertEquals('<h2>Hello</h2>', $client->getResponse()->getContent());
    }
}
```

## 关于服务

具体参考：<https://symfony.com/doc/current/service_container.html>

这里是一个**文件上传服务**示例：

```php
<?php

namespace App\Service;

use Symfony\Component\HttpFoundation\File\Exception\FileException;
use Symfony\Component\HttpFoundation\File\UploadedFile;

class FileUploader
{
    private $targetDirectory;
    private $targetPath;
    private $finalHost;
    private $finalUrl;

    public function __construct(string $targetDirectory, ?string $targetHost)
    {
        // TODO: 如果配置了 OSS 文件服务，需要给予 创建目录+写入+读取 的权限
        // DONE: 需要根据配置文件的 文件上传域名 判断该上传至哪个文件夹（在同一个服务器，但与代码分开存放
        if (empty($targetDirectory)) {
            $targetDirectory = dirname(__FILE__, 3) . '/public/static/upload';
        }

        $targetDirectory = $targetDirectory . '/' . date('Y') . '/' . date('m') . '/';
        $targetDirectory = preg_replace('#\/{2,}#', '/', $targetDirectory);
        if (!file_exists($targetDirectory)) {
            mkdir($targetDirectory, 0777, true);
        }
        $this->targetDirectory = $targetDirectory;

        $this->finalHost = str_replace(dirname(__FILE__, 3) . '/public', '',  $this->targetDirectory);
        if (!empty($targetHost)) {
            $this->finalHost = $targetHost . '/' . $this->finalHost;
        }
    }

    // TODO: 多文件上传

    /**
     * 文件上传
     *
     * @param UploadedFile $file
     * @return array
     */
    public function upload(UploadedFile $file): array
    {
        $originalFilename = pathinfo($file->getClientOriginalName(), PATHINFO_FILENAME);

        $filename = $originalFilename . '.' . $file->guessExtension();
        $safeFileName = md5_file($file) . '.' . $file->guessExtension();

        try {
            $this->targetPath = $this->targetDirectory . $safeFileName;

            if (!file_exists($this->targetPath)) {
                $file->move($this->getTargetDirectory(), $safeFileName);
            }

            $this->finalUrl = preg_replace('#\/{2,}#', '/', $this->finalHost . '/' . $safeFileName);

            $fileSize = round(filesize($this->targetPath) / 1024, 2);
            $mimeType = mime_content_type($this->targetPath);

            $info = [
                'file_name' => $filename,
                'file_path' => $this->targetPath,
                'file_url' => $this->finalUrl,
                'file_size' => $fileSize,
                'mime_type' => $mimeType,
            ];

            $imageSize = getimagesize($this->targetPath);
            if ($imageSize !== false) {
                $info['width'] = $imageSize[0];
                $info['height'] = $imageSize[1];
            }
        } catch (FileException $e) {
            throw new FileException($e->getMessage());
        }

        return $info;
    }

    /**
     * 获取-上传文件目录
     *
     * @return string
     */
    public function getTargetDirectory()
    {
        return $this->targetDirectory;
    }
}
```

需要注意的是，每注册一个服务，需要去`config/service.yaml`文件“挂个名”：

```yaml
services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true      # Automatically injects dependencies in your services.
        autoconfigure: true # Automatically registers your services as commands, event subscribers, etc.

    # ...

    App\Service\:
        resource: '../src/Service'

    App\Service\FileUploader:
        # 构造函数需要的参数由这里注入
        arguments:
            $targetDirectory: '%env(resolve:UPLOAD_DIR)%'
            $targetHost: '%env(resolve:UPLOAD_HOST)%'
```


##  Doctrine ORM 疑惑 - 1

今天遇到一个问题。

```php
$project = $projectRepository->find($id);

if (!$project) {
    throw new Exception('该项目不存在！ID: ' . $id);
}

$data = json_decode($request->getContent(), true);

$entityManager = $this->getDoctrine()->getManager();

foreach ($project->getSettlements() as $settlement) {
    $project->removeSettlement($settlement);
}

foreach ($data as $settlement) {
    $projectSettlement = new ProjectSettlement();
    $projectSettlement->setDelimit($settlement['delimit']);
    $projectSettlement->setDate(strtotime($settlement['date']));
    $projectSettlement->setPercent($settlement['percent']);
    $projectSettlement->setAmount($settlement['amount']);
    // $projectSettlement->setProject($project);

    $project->addSettlement($projectSettlement);
    $entityManager->persist($projectSettlement);
}

$entityManager->flush();

$result = $project->jsonSerialize();
$result['settlements'] = $project->getSettlements()->toArray();
return new JsonResponse($result['settlements']);
```

预计是返回一个数组，但返回了一个对象：

```json
{
    "2": {
        "id": 117,
        "date": "2020-10-10 10:10:10",
        "delimit": "",
        "percent": "23.8",
        "amount": "3444.23",
        "create_at": "2020-09-23 09:35:16"
    },
    "3": {
        "id": 118,
        "date": "",
        "delimit": "结算条件：当金额达到百分之三十时结算",
        "percent": "23.8",
        "amount": "3444.23",
        "create_at": "2020-09-23 09:35:16"
    }
}
```

数据库是正确的，但是返回的形式完全错了。
后查找原因可能是 Doctrine 懒加载的原因，在最后获取 settlement 时，采用循环而不是`toArray()`方法，即可获取预期结果：

```php
$result = $project->jsonSerialize();
foreach ($project->getSettlements() as $settlement) {
    $result['settlements'][] = $settlement->jsonSerialize();
}
```

## Doctrine ORM 疑惑 - 2

还是[上一个问题](#Doctrine ORM 疑惑 - 1)，因为设置了主表 project 与子表 project_settlement 的一对多关联，在给添加 project 添加 settlement 时，前面使用的是`$project->addSettlement($projectSettlement);`方法，后来想想`ProjectSettlement`模型中有`setProject()`方法，便尝试使用`$projectSettlement->setProject($project);`形式，结果在输出时`settlements`返回空，但数据库是正常添加的。


