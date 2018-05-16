---
layout: post
title: mysql中文乱码问题
date: 2017-09-12 00:00:00 +0300
description: mysql保存中文数据乱码问题处理 # Add post description (optional)
img: software.jpg # Add image post (optional)
tags: [mysql, 乱码] # add tag
---

mysql是我们项目中非常常用的数据型数据库。但是因为我们需要在数据库保存中文字符，所以经常遇到数据库乱码情况。下面就来介绍一下如何彻底解决数据库中文乱码情况。


# 创建库时采用UTF-8 编码

{% highlight mysql %}
  CREATE DATABASE db_name DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
{% endhighlight%}

# 程序链接时采用utf-8 方式

{% highlight java %}
   jdbc:mysql://192.168.31.199:3306/TT?useUnicode=true&characterEncoding=utf-8
{% endhighlight%}
