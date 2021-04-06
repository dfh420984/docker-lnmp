## Docker LNMP 3.0

Docker LNMP 可以构建出基于 Docker 的 PHP 开发环境，其优势有在短时间内随意构建不同版本的相关服务、环境统一分布在不同服务器等，使开发者能够更专注于开发业务本身。

##### 产品特色

* 灵活切换适合国内的源（Apt-get、PHP Composer）
* 组件精简易懂，学习、测试环境、生产环境均适合
* 可能是最易用的计划任务（安装在 PHP 组件里）
* 良好的扩展性
* 非常合理的结构划分（吐槽 Laradock）

##### 版本及组件

* 当前版本：3.0
* 自带组件：PHP/FPM 7.2、Nginx latest、Mysql 5.6、Redis 4.0

##### 目录结构

    docker-lnmp
    |----/build                  镜像构建目录
    |----/work                   持久化目录
    |--------/components/        组件库
    |------------/component      组件，包括了数据、配置文件、日志等持久化数据
    |-----------------/config    组件的配置目录
    |-----------------/log       组件的日志目录
    |--------/wwwroot            WEB 文件目录
    |----/.env-example           配置文件
    |----/docker-compose.yml     docker compose 配置文件

#### 开始安装

没有安装 Docker 的同学移步 [安装教程](https://github.com/exc-soft/docker-lnmp#安装-docker-及相关工具)

    cd ~/
    git clone https://github.com/exc-soft/docker-lnmp.git

    cd docker-lnmp
    cp .env-example .env

    # 配置数据库密码、时区、端口等
    vim .env

    # 构建镜像并启动容器
    sudo docker-compose up --build -d

启动成功访问 http://localhost 即可

#### 可能遇到的问题

##### 常用操作命令

    # 查看当前启动的容器
    sudo docker ps
    
    # 启动部分服务在后边加服务名，不加表示启动所有，-d 表示在后台运行
    sudo docker-compose up  -d [nginx|php|mysql|redis]
    
    # 停止和启动类似
    sudo docker-compose stop [nginx|php|mysql|redis]

    # 删除所有未运行的容器
    sudo docker rm $(docker ps -a -q)

    # 删除所有镜像，-f 可以强制删除
    sudo docker rmi $(docker images -q)

##### 修改镜像文件怎么处理？
    
    # 比如在 php 里新增一个扩展
    # 1、更改对应的 docker-lnmp/build/php/dockerfile
    # 2、重新构建镜像
    sudo docker-compose build php
    

##### 如何设置开机启动服务？

    # 编辑开机启动文件，写入  cd /home/your/docker-lnmp && compose up -d
    # 注意这里不用 sudo，本身是使用 root 运行的
    sudo vim /etc/rc.local

    # 重启测试
    sudo reboot

##### 如何在 php 里连接 Mysql 和 Redis？

    <?php
        // 连接 Mysql
        $user = "root";
        $pass = "DockerLNMP";
        $dbh = new PDO('mysql:host=mysql;dbname=mysql', $user, $pass);
        
        foreach($dbh->query('SELECT * from user') as $row) {
            print_r($row);
        }

        // 连接 Redis
        $redis = new Redis();
        $redis->connect('redis', 6379);
        $redis->set("test-key","hello");
        echo "Stored string in redis:: " . $redis->get("test-key");
    ?>

##### 如何使用 PHP 组件里的计划任务？

直接在 /work/components/php/cron.d 新建 crontab 文件或者使用自带的 task 就行了，比如做数据库备份计划可以将备份的文件保存到 cron.d 同级 backup 目录下面，其中有几个坑都趟平了可以看下 task 文件里的注释说明，另外不用担心容器重启后计划任务停止，只要容器在运行你的任务就在运行。

##### 其他的坑

如果需要升级某些组件的版本需要注意载入对应版本的配置文件，修改对应的配置信息，比如 redis.conf 默认配置的出口 ip 为 127.0.0.1，这样的话 php 的容器是连不上的，需要修改成 0.0.0.0，另外也要注意修改对应的 log path。

## 安装 Docker 及相关工具

##### 1、安装 docker （2种方式都可以）
    
    # 1、如果你是 ubuntu 系统，可以尝试执行下如下命令安装，有问题再试下方法2
    apt install docker.io
    
    # 2、参考 daocloud 提供的文档
    # 注意按照文档如果执行类似 install docker-ce=17.03.1* 出错，执行 install docker-ce 即可
    https://download.daocloud.io/Docker_Mirror/Docker
    
    

##### 2、安装 docker-compose
    
    # 注意：你如果用的是非 root 用户，执行 curl 会提示没权限写入 /usr/local/bin 目录，可以先写入当前目录，再使用 sudo mv 过去
    curl -L https://get.daocloud.io/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

##### 3、启动 Docker

    sudo service docker start
    sudo docker info    

##### 4、配置 DockerHub 加速器（这简直是一定的）

阿里云加速器：每个人有对应的加速地址，访问 https://dev.aliyun.com ->【管理中心】-> 【DockerHub 镜像站点】配置加速器

DaoCloud 加速器：http://guide.daocloud.io/dcs/daocloud-9153151.html

腾讯云加速器：https://www.qcloud.com/document/product/457/7207

## License
MIT
