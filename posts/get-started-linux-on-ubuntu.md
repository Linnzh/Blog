# 在 Linux 系统学习：以 Ubuntu 18.04 为例

## Reference

- [HRB 的 GitHub](https://symfony.com/doc/current/doctrine.html)

-------------------------------

## 配置 vim

添加**行号显示**、**粘贴模式**等

## 配置终端

参考[秋月无边]()

## 编译 nginx

### nginx 服务器配置文件参考

```conf
server {
    listen      80;
    server_name example.linnzh.com;

    root    /home/linnzh/get-started-symfony/public;
    charset utf-8;

    index  index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # 开启PHP-CGI解析
    location ~ \.php$ {
       fastcgi_pass   127.0.0.1:9000; #unix:/dev/shm/php-fpm-7.3.22.sock;
       fastcgi_index  index.php;
       fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
       include        fastcgi_params;
    }
}
```

## 编译 PHP 7

### 安装 composer

## 安装 Docker 与 Docker Compose
