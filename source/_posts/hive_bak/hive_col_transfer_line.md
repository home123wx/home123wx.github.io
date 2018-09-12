---
title: Hive列转行
tags: [sql,hive]
date: 2017-12-05
categories: Hive
---

#### 列转行

hive中由多行数据转换成一行记录

test表数据：
```
id  name
--  -----
1	aaa
1	bbb
1	ccc
1	ddd
1	eee
1	fff
1	ggg
1	hhh
1	iii
```

##### 使用collect_set()函数聚合id相同的name

```
select id, collect_set(name) from test group by id;
```

*结果为：*

```
1	["aaa","bbb","ccc","ddd","eee","fff","ggg","hhh","iii"]
```

##### 结合concat_ws()函数拼接成字符串

```
select id, concat_ws(",", collect_set(name)) from test group by id;
```

*结果为：*

```
1	aaa,bbb,ccc,ddd,eee,fff,ggg,hhh,iii
```