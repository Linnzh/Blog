# MySQL 安装

## Reference

- [MySQL 8.0](http://ftp.iij.ad.jp/pub/db/mysql/Downloads/MySQL-8.0/)

## Windows 10

1. 解压地址：`C:\Users\Administator\mysql-8.0.16`
2. 打开控制台并按以下操作输入：
    
    ```sh
    # 进入解压地址的`bin`目录
    cd C:\Users\Administator\mysql-8.0.16\bin

    # 初始化数据库并将初始密码打印在控制台
    # 注意控制台输出的信息，MySQL 的初始密码会被输出在控制台中
    # 形如：root@localhost: jkeDoHgY0<fR
    mysqld --initialize --console

    # 注册 MySQL 服务
    mysqld --install mysql8

    # 启动 MySQL 服务
    net start mysql8

    # 使用初始密码以 root 身份进入数据库
    mysql -u root -p

    # 修改密码插件及密码
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

    # 退出 MySQL
    quit;
    ```

3. 将 MySQL 的`bin`目录所在路径添加至系统的 PATH 环境变量，以便使用命令行进入数据库

------------------------------------------------------------------------------------------------------------------------

## WSL 2


