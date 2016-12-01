# PHP开发环境使用指南

我司开发环境主要使用：
* jmaple/docker-php-nginx（集成了 PHP-FPM 7.0.11 + NGINX 1.10.2）
* mysql:5.7.12
* redis:3.0.7-alpine

## 使用要求

* [Docker 1.10+](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

## 推荐目录结构

```
-dev_env_config           // 开发环境相关配置文件
  |-docker-compose.yml   // docker-compose 配置文件
  |-mysql.d              // mysql 配置文件目录
    |-my.cnf             // mysql 配置文件
  |-nginx                // nginx 配置文件目录
    |-nginx.conf         // nginx 配置文件
    |-conf.d             // nginx 配置文件目录
      |-default.conf
  |-php                  // php 配置文件目录
    |-php.ini
  |-php-fpm.d            // php-fpm 配置文件目录
    |-www.conf

-www                     // 项目目录
  |-project1
  |-project2
  |-...
```

## 启动容器

在dev_env_config目录（此目录必须包含docker-compspe.yml文件）下执行命令

```
docker-compose up -d
```

或者在任意目录下执行命令

```
docker-compose -f /YOUR-PATH/docker-compose.yml up -d
```

此时docker-compose会自动检查各个容器是否存在、是否最新，然后自动启动各个容器

[docker 常用命令](common/docker.md)

## dphp命令

为了让大家更方便的使用容器来开发，故提供一种可以更便捷的使用docker exec -ti <DOCKER-PHP-NGINX> 容器的命令 **dphp**

### 配置方法

在Linux系统里创建一个shell脚本，dphp.sh，我们把它放在/opt/目录下

脚本内容如下（可自己根据实际需求定制修改）：

```
#!/bin/sh
VIF=$(pwd|grep "www" | wc -l)
VAR=$(pwd)
if test $VIF = 0; then
  echo 'only use dphp in project folder'
else
  echo ${pwd}
# 请把下面的<YOUR-CONTAINER-NAME>替换成你实际的docker-php-nginx容器名，可使用docker ps查看
  docker exec -ti <YOUR-CONTAINER-NAME> "/data/www/""${VAR##*/}/""$@"
fi
```

配置环境变量

```
vi /etc/bashrc
```

在最后一行加上

```
alias dphp='/opt/dphp.sh'
```

重新登录终端，dphp即生效

### 使用要点

- 在项目目录下比如/www/project1执行dphp yii，即相当于docker exec -ti <DOCKER-PHP-NGINX> /data/www/project1/yii
- 可以在后面加参数 比如 dphp yii migrate/create customer_status
- 在项目目录下使用才有效，此处限制了项目目录的上一层文件夹名为www，可自行修改shell脚本定制
