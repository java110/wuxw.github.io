---
layout: post
title: Mysql数据库及表空间占用信息统计
date: 2017-09-12 00:00:00 +0300
description: Mysql数据库及表空间占用信息统计。 # Add post description (optional)
img: workflow.jpg # Add image post (optional)
tags: [mysql, 表空间] # add tag
---

# 1、mysql中查看各表的大小

这里用到一个表, information_schema.tables；对应主要字段含义如下

{% highlight mysql %}

ABLE_SCHEMA : 数据库名
TABLE_NAME：表名
ENGINE：所使用的存储引擎
TABLES_ROWS：记录数
DATA_LENGTH：数据大小
INDEX_LENGTH：索引大小

{% endhighlight%}

按记录数据统计：
{% highlight mysql %}
select table_schema,table_name,table_rows from tables order by table_rows desc;  
{% endhighlight %}

# 2、查询所有数据库占用磁盘空间大小的SQL语句

{% highlight mysql %}
select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,  
concat(truncate(sum(index_length)/1024/1024,2),' MB') as index_size  
from information_schema.tables  
group by TABLE_SCHEMA  
order by data_length desc;    
{% endhighlight %}

# 3、查询单个库中所有表磁盘占用大小的SQL语句


{% highlight mysql %}
select TABLE_NAME, concat(truncate(data_length/1024/1024,2),' MB') as data_size,  
concat(truncate(index_length/1024/1024,2),' MB') as index_size  
from information_schema.tables where TABLE_SCHEMA = 'sor'  
group by TABLE_NAME  
order by data_length desc;    
{% endhighlight %}

# 4、查看一个库中的使用情况

{% highlight mysql%}
SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name',  
 CONCAT(ROUND(table_rows/1000000,4),'M') AS 'Number of Rows',   
 CONCAT(ROUND(data_length/(1024*1024*1024),4),'G') AS 'Data Size',   
 CONCAT(ROUND(index_length/(1024*1024*1024),4),'G') AS 'Index Size',   
 CONCAT(ROUND((data_length+index_length)/(1024*1024*1024),4),'G') AS 'Total'   
FROM information_schema.TABLES   
WHERE table_schema LIKE 'src';  
{% endhighlight %}
