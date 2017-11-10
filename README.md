# Nginx PHP MySQL [![Build Status](https://travis-ci.org/h136799711/docker-init.svg?branch=master)](https://travis-ci.org/h136799711/docker-init) [![GitHub version](https://badge.fury.io/gh/nanoninja%2Fdocker-nginx-php-mysql.svg)](https://badge.fury.io/gh/nanoninja%2Fdocker-nginx-php-mysql)

在Docker环境下运行 Nginx,PHP-FPM,Composer,MySQL 和 PHPMyAdmin

## 概述

1. [安装预先软件](#install-prerequisites)
    
    在安装项目之前，确保满足以下先决条件。

2. [克隆项目](#clone-the-project)

    我们将从GitHub上下载代码。

3. [nginx的SSL证书配置](#configure-nginx-with-ssl-certificates) [可选]

    我们将生成和配置nginx的SSL证书在运行服务器之前。

4. [配置 Xdebug](#configure-xdebug) [可选]
    我们将为IDE(PHPStorm 或 Netbeans) 配置 Xdebug
5. [运行应用](#run-the-application)
    到这一步，已经做好所有准备工作了.

6. [使用 Makefile](#use-makefile) [可选]

   在开发的时候，你可以用` makefile `做重复的操作。

7. [使用 Docker 命令](#use-docker-commands)

    运行时，你可以使用Docker命令做重复的操作。

___

## 安装先决软件
目前，这个项目主要是为unix`(Linux/MacOS)`创建的。也许它可以在Windows上工作。
最重要需要安装的软件 :

* [Git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

检查 `docker-compose` 是否已经安装了，通过输入以下命令: 

```sh
which docker-compose
```
检查Docker Compose 兼容性:
 - [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

以下是可选的但是可以使开发更方便

```sh
which make
```
在 Ubuntu and Debian 这些是内置可用的。在其它发布版本下，你可能需要单独安装GNU C++ 编译器。

```sh
sudo apt install build-essential
```

### 使用的镜像

* [Nginx](https://hub.docker.com/_/nginx/)
* [MySQL](https://hub.docker.com/_/mysql/)
* [PHP-FPM](https://hub.docker.com/r/nanoninja/php-fpm/)
* [Composer](https://hub.docker.com/_/composer/)
* [PHPMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
* [Generate Certificate](https://hub.docker.com/r/jacoelho/generate-certificate/)

你应该额外小心的安装第三方Web服务器，例如: mysql 或 nginx.

这些项目使用以下端口 :

| 服务       | 端口 |
|------------|------|
| MySQL      | 8989 |
| PHPMyAdmin | 8080 |
| Nginx      | 8000 |
| Nginx SSL  | 3000 |

---

## 克隆项目

安装 [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git), 下载它并且按照说明安装 : 

```sh
git clone https://github.com/h136799711/docker-init.git
```

进入项目目录：

```sh
cd docker-nginx-php-mysql
```

### 项目结构

```sh
.
├── Makefile
├── README.md
├── data
│   └── db
│       ├── dumps
│       └── mysql
├── doc
├── docker-compose.yml
├── etc
│   ├── nginx
│   │   ├── default.conf
│   │   └── default.template.conf
│   ├── php
│   │   └── php.ini
│   └── ssl
└── web
    ├── app
    │   ├── composer.json
    │   ├── composer.json.dist
    │   ├── phpunit.xml.dist
    │   ├── src
    │   │   └── Foo.php
    │   └── test
    │       ├── FooTest.php
    │       └── bootstrap.php
    └── public
        └── index.php
```

---

## 使用SSL证书配置Nginx

你可以修改 主机名称通过编辑 `.env` 文件
如果你修改了 主机名 ，不要忘记了把它加入到 `/etc/hosts` 文件.

1. 生成 SSL certificates

    ```sh
    source .env && sudo docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=$NGINX_HOST" jacoelho/generate-certificate
    ```

2. 配置 Nginx

    不要修改 `etc/nginx/default.conf` 文件, 它会被  `etc/nginx/default.template.conf` 文件的内容覆盖

    编辑nginx文件 `etc/nginx/default.template.conf` 并且取消注释 SSL 服务小节 :

    ```sh
    # server {
    #     server_name ${NGINX_HOST};
    #
    #     listen 443 ssl;
    #     fastcgi_param HTTPS on;
    #     ...
    # }
    ```

---

## 配置 Xdebug

如果你用的是除了 [PHPStorm](https://www.jetbrains.com/phpstorm/) 或 [Netbeans](https://netbeans.org/), 请查看 Xdebug 文档的 [remote debugging](https://xdebug.org/docs/remote) 小节内容.

For a better integration of Docker to PHPStorm, use the [documentation](https://github.com/h136799711/docker-init/blob/master/doc/phpstorm-macosx.md).

1. 获取本地ip地址 :
    linux
    ```sh
    sudo ifconfig
    ```
    windows
    ```sh
    ipconfig
    ```
    

2. 编辑php文件 `etc/php/php.ini` 并注释或取消注释你所需要的.

3. 设置 `remote_host` 参数值为你的本地IP :

    ```sh
    xdebug.remote_host=192.168.0.1 # 你本地机子的ip
    ```
---

## 运行应用

1. 拷贝composer 配置文件 ：
    ```sh
    cp web/app/composer.json.dist web/app/composer.json
    ```

2. 启动应用 :

    ```sh
    sudo docker-compose up -d
    ```

    **Please wait this might take a several minutes...**

    ```sh
    sudo docker-compose logs -f # Follow log output
    ```

3. 打开你喜欢的浏览器 :

    * [http://localhost:8000](http://localhost:8000/)
    * [https://localhost:3000](https://localhost:3000/) ([HTTPS](#configure-nginx-with-ssl-certificates) not configured by default)
    * [http://localhost:8080](http://localhost:8080/) PHPMyAdmin (username: dev, password: dev)

4. 暂停和清理服务

    ```sh
    sudo docker-compose down -v
    ```

---

## 使用Makefile

开发的时候，你可以使用 进行下列操作  [Makefile](https://en.wikipedia.org/wiki/Make_(software)) :

| 名称          | 描述                                |
|---------------|--------------------------------------------|
| apidoc        | Generate documentation of API              |
| clean         | Clean directories for reset                |
| code-sniff    | Check the API with PHP Code Sniffer (PSR2) |
| composer-up   | Update PHP dependencies with composer      |
| docker-start  | Create and start containers                |
| docker-stop   | Stop and clear all services                |
| gen-certs     | Generate SSL certificates for `nginx`      |
| logs          | Follow log output                          |
| mysql-dump    | Create backup of whole database            |
| mysql-restore | Restore backup from whole database         |
| test          | Test application with phpunit              |

### 例子

启动应用 : 

```sh
sudo make docker-start
```

显示帮助 :

```sh
make help
```

---

## 使用Docker命令

### 通过Composer 安装模块

```sh
sudo docker run --rm -v $(pwd)/web/app:/app composer require symfony/yaml
```

### 使用 composer 更新 PHP依赖

```sh
sudo docker run --rm -v $(pwd)/web/app:/app composer update
```

### 生成PHP API 文档

```sh
sudo docker-compose exec -T php ./app/vendor/bin/apigen generate app/src --destination ./app/doc
```

### 使用PHPUnit 测试 PHP 应用

```sh
sudo docker-compose exec -T php ./app/vendor/bin/phpunit --colors=always --configuration ./app/
```

### 使用[PSR2](http://www.php-fig.org/psr/psr-2/)校验标准代码

```sh
sudo docker-compose exec -T php ./app/vendor/bin/phpcs -v --standard=PSR2 ./app/src
```

### 检查已安装PHP扩展

```sh
sudo docker-compose exec php php -m
```

### 处理数据库

#### MySQL shell 访问

```sh
sudo docker exec -it mysql bash
```

和

```sh
mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
```

#### 备份数据库

```sh
mkdir -p data/db/dumps
```

```sh
source .env && sudo docker exec $(shell docker-compose ps -q mysqldb) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
```

或

```sh
source .env && sudo docker exec $(shell docker-compose ps -q mysqldb) mysqldump test -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/test.sql"
```

#### 还原数据库

```sh
source .env && sudo docker exec -i $(sudo docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
```

#### 链接Mysql 使用 [PDO](http://php.net/manual/en/book.pdo.php)

```php
<?php
    try {
        $dsn = 'mysql:host=mysql;dbname=test;charset=utf8;port=3306';
        $pdo = new PDO($dsn, 'dev', 'dev');
    } catch (PDOException $e) {
        echo $e->getMessage();
    }
?>
```

---