---
title: 使用over()函数结合PARTITION BY以指定字段分区
tags: [sql,hive]
date: 2017-12-06
categories: Hive
---

#### over()函数和PARTITION BY结合使用

``` sql
SELECT 
  item_id,
  item_name
FROM
  (SELECT 
    item_id,
    item_name,
    sort_item, 
    row_number () over (PARTITION BY item_id ORDER BY sort_item DESC) AS rm 
  FROM
    table_name) AS t
WHERE t.rm = 1
```

* 以item_id分区，所有item_id相同的被放入一个分区中
* 分区内使用row_number()对每一行做标识
* 根据sort_item字段进行排序
* 获取每个分区中的第一数据
