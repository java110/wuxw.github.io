---
layout: post
title: docker 构建mysql 创建库，创建表，导入数据
date: 2017-09-12 00:00:00 +0300
description: 在用docker创建mysql容器的时，有时候我们期望容器启动后数据库和表已经自动建好，初始化数据也已自动录入，也就是说容器启动后我们就能直接连上容器中的数据库，使用其中的数据了。 # Add post description (optional)
img: how-to-start.jpg # Add image post (optional)
tags: [docker, mysql] # add tag
---
在用docker创建mysql容器的时，有时候我们期望容器启动后数据库和表已经自动建好，初始化数据也已自动录入，也就是说容器启动后我们就能直接连上容器中的数据库，使用其中的数据了。

下面我们一起来看看

## 1、制作sql文件 create_sql.sql 内容如下：

{% highlight mysql %}

CREATE USER 'TT'@'%' IDENTIFIED BY 'TT@12345678';

GRANT ALL ON *.* TO 'TT'@'%';

create database `TT` default character set utf8 collate utf8_general_ci;

use TT;

-- c_status

CREATE TABLE c_status(
    id INT NOT NULL AUTO_INCREMENT KEY COMMENT 'id',
    status_cd VARCHAR(4) NOT NULL COMMENT '状态',
    `name` VARCHAR(50) NOT NULL COMMENT '名称',
    description VARCHAR(200) COMMENT '描述',
    create_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
);

INSERT INTO c_status(status_cd,NAME,description) VALUES('1','无效的，不在用的','无效的，不在用的');
INSERT INTO c_status(status_cd,NAME,description) VALUES('0','有效的，在用的','有效的，在用的');
INSERT INTO c_status(status_cd,NAME,description) VALUES('S','保存成功','保存成功');
INSERT INTO c_status(status_cd,NAME,description) VALUES('D','作废订单','作废订单');
INSERT INTO c_status(status_cd,NAME,description) VALUES('E','错误订单','错误订单');
INSERT INTO c_status(status_cd,NAME,description) VALUES('NE','通知错误订单','通知错误订单');
INSERT INTO c_status(status_cd,NAME,description) VALUES('C','错误订单','错误订单');


{% endhighlight}

## 2、制作dockerfile 文件

{% highlight java %}

# Docker image of java110 mysql
# VERSION 0.0.1
# Author: jack wu

FROM mysql:5.6

#作者
MAINTAINER jackWu <928255095@qq.com>

#定义工作目录
ENV WORK_PATH /usr/local/work

#定义会被容器自动执行的目录
ENV AUTO_RUN_DIR /docker-entrypoint-initdb.d

#定义sql文件名
ENV FILE_0 create_sql.sql


#定义shell文件名
ENV INSTALL_DATA_SHELL setup.sh

#创建文件夹
RUN mkdir -p $WORK_PATH

#把数据库初始化数据的文件复制到工作目录下
COPY ./$FILE_0 $WORK_PATH/


#把要执行的shell文件放到/docker-entrypoint-initdb.d/目录下，容器会自动执行这个shell
COPY ./$INSTALL_DATA_SHELL $AUTO_RUN_DIR/

#给执行文件增加可执行权限
RUN chmod a+x $AUTO_RUN_DIR/$INSTALL_DATA_SHELL

{% endhighlight %}

3、制作启动脚本setup.sh

{% highlight shell %}

#!/bin/bash
mysql -uroot -p$MYSQL_ROOT_PASSWORD <<EOF
source $WORK_PATH/$FILE_0;

{% endhighlight %}

4、构建时所需要手工运行的命令

{% highlight shell %}

docker build -t java110/docker-mysql .

docker run -ti --name mysql_test -e MYSQL_ROOT_PASSWORD=123456 -p3306:3306 -idt java110/docker-mysql:latest

docker logs -f mysql_test

docker exec -it mysql_test /bin/bash

mysql -uroot -p123456

{% endhighlight %}
