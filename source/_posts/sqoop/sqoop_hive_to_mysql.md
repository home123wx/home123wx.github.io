---
title: 使用Sqoop同步 mysql <--> hive 数据
tags: [sqoop,hive,mysql]
date: 2017-12-01
categories: [Hive,Sqoop]
---


#### hive 到 mysql 同步

``` shell 
sqoop export 
    --connect jdbc:mysql://[ip]:[port]/db_name?characterEncoding=utf-8
    --username name
    --password passwd
    --table table_name
    --input-fields-terminated-by "\t" (hive表中字段之间使用的分割符)
    --input-lines-terminated-by "\n" (hive表中行与行之间使用的分隔符)
    --export-dir [hdfs存储位置] (通过 show create table table_name可以查看)
    --update-key key_id (mysql表中对应的主键)
    --update-mode allowinsert (更新模式[allowinsert, updateonly])
```

#### mysql 到 hive 同步

``` shell 
sqoop import 
    --connect jdbc:mysql://[ip]:[port]/db_name
    --username username
    --password passwd
    --table srcMysqltableName
    --fields-terminated-by "\t" (构建后hive表中的字段分隔符)
    --hive-drop-import-delims (去掉hive默认的行分隔符"\n")
    --hive-import (导入数据到hive)
    --hive-overwrite
    --create-hive-table 
    --hive-table destHivetableName
    --delete-target-dir
```
