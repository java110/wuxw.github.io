---
layout: post
title: "spring boot 测试环境生产环境不同配置"
date: 2017-04-06 13:32:20 +0300
description: 以前我们用weblogic时，数据源配置等是通过weblogic jndi 配置，如今微服务时代我们喜欢轻量级的spring boot 架构系统，那么数据源信息该如何配置呢？ # Add post description (optional)
img:  # Add image post (optional)
---
以前我们用weblogic时，数据源配置等是通过weblogic jndi 配置，如今微服务时代我们喜欢轻量级的spring boot 架构系统，那么数据源信息该如何配置呢？我们知道spring boot 是通过application.yml 文件 或者 application.properties文件 配置的，我们不想在每次启动的时候修改application 文件，那么如何才能实现呢？ 下面我们一起探讨：

# 1、我们在项目中为每一个环境增加配置文件

我们在项目中，根据每个环境增加相应的 application-xxx.yml 文件，配置相应环境的信息。 如开发环境 application-dev.yml ：

{% highlight java %}

jedis:
  pool:
    config:
      maxTotal: 100
      maxIdle: 20
      maxWaitMillis: 20000
    host: 192.168.31.199
    port: 6379

eureka:
  instance:
    leaseRenewalIntervalInSeconds: 10
    leaseExpirationDurationInSeconds: 30
  client:
    serviceUrl:
      defaultZone: http://192.168.31.199:8761/eureka/
      #defaultZone: http://localhost:8761/eureka/
server:
  port: 8001
  tomcat:
    uri-encoding: UTF-8

spring:
  http:
    encoding:
      charset: UTF-8
      enabled: true
      force: true
  application:
    name: center-service
  redis:
    database: 0
    host: 192.168.31.199
    port: 6379
    pool:
      max-active: 300
      max-wait: 10000
      max-idle: 100
      min-idle: 0
      timeout: 0
  datasource:
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    minIdle: 5
    validationQuery: SELECT 1 FROM DUAL
    initialSize: 5
    maxWait: 60000
    filters: stat,wall,log4j
    poolPreparedStatements: true
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://192.168.31.199:3306/TT?useUnicode=true&characterEncoding=utf-8
    maxPoolPreparedStatementPerConnectionSize: 20
    password: TT@12345678
    testOnBorrow: false
    testWhileIdle: true
    minEvictableIdleTimeMillis: 300000
    timeBetweenEvictionRunsMillis: 60000
    testOnReturn: false
    driverClassName: com.mysql.jdbc.Driver
    maxActive: 20
    username: TT

#============== kafka ===================
kafka:
  consumer:
    zookeeper:
      connect: 192.168.31.199:2181
    servers: 192.168.31.199:9092
    enable:
      auto:
        commit: true
    session:
      timeout: 6000
    auto:
      commit:
        interval: 100
      offset:
        reset: latest
    topic: test
    group:
      id: notifyBusinessStatus
    concurrency: 10

  producer:
    zookeeper:
      connect: 192.168.31.199:2181
    servers: 192.168.31.199:9092
    retries: 0
    batch:
      size: 4096
    linger: 1
    buffer:
      memory: 40960

{% endhighlight %}

如生产环境application-prod.yml
{% highlight java %}

jedis:
  pool:
    config:
      maxTotal: 100
      maxIdle: 20
      maxWaitMillis: 20000
    host: 135.192.86.200
    port: 6379

eureka:
  instance:
    leaseRenewalIntervalInSeconds: 10
    leaseExpirationDurationInSeconds: 30
  client:
    serviceUrl:
      defaultZone: http://135.192.86.200:8761/eureka/
      #defaultZone: http://localhost:8761/eureka/
server:
  port: 8001
  tomcat:
    uri-encoding: UTF-8

spring:
  http:
    encoding:
      charset: UTF-8
      enabled: true
      force: true
  application:
    name: center-service
  redis:
    database: 0
    host: 135.192.86.200
    port: 6379
    pool:
      max-active: 300
      max-wait: 10000
      max-idle: 100
      min-idle: 0
      timeout: 0
  datasource:
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    minIdle: 5
    validationQuery: SELECT 1 FROM DUAL
    initialSize: 5
    maxWait: 60000
    filters: stat,wall,log4j
    poolPreparedStatements: true
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://135.192.86.200:3306/TT?useUnicode=true&characterEncoding=utf-8
    maxPoolPreparedStatementPerConnectionSize: 20
    password: TT@12345678
    testOnBorrow: false
    testWhileIdle: true
    minEvictableIdleTimeMillis: 300000
    timeBetweenEvictionRunsMillis: 60000
    testOnReturn: false
    driverClassName: com.mysql.jdbc.Driver
    maxActive: 20
    username: TT

#============== kafka ===================
kafka:
  consumer:
    zookeeper:
      connect: 135.192.86.200:2181
    servers: 135.192.86.200:9092
    enable:
      auto:
        commit: true
    session:
      timeout: 6000
    auto:
      commit:
        interval: 100
      offset:
        reset: latest
    topic: test
    group:
      id: notifyBusinessStatus
    concurrency: 10

  producer:
    zookeeper:
      connect: 135.192.86.200:2181
    servers: 135.192.86.200:9092
    retries: 0
    batch:
      size: 4096
    linger: 1
    buffer:
      memory: 40960

{% endhighlight %}

#2、在 application.yml 中引入

在启动相应环境时修改成相应环境 如开发环境 dev

{% highlight java %}

spring:
  profiles:
    active: dev

{% endhighlight %}

#3、启动脚本控制

如果你是要根据启动脚本来控制 则在启动时加入-Dspring.profiles.active=dev

{% highlight java %}

java -jar -Dspring.profiles.active=dev $1 XXX.jar

{% endhighlight %}
