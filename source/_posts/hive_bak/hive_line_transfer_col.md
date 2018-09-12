---
title: Hive行转列
tags: [sql,hive]
date: 2017-11-29
categories: Hive
---

#### 行转列
把Hive中，一行的数据转换成多条数据

test表数据：
```
a	aaa
b	bbb
c	ccc
d	ddd
e	eee
f	fff
g	ggg
h	hhh
i	iii
```

##### 构建使用分隔符分割 (",") 的数据
``` sql
SELECT CONCAT_WS(',', '1', '2', '3', '4', '5', '6', '7', '8', '9') FROM test;
```

数据为：
```
1,2,3,4,5,6,7,8,9
```

##### 使用分割函数 split
``` sql
SELECT split(CONCAT_WS(',', '1', '2', '3', '4', '5', '6', '7', '8', '9'), ",") FROM test;
```

数据为: 
```
["1","2","3","4","5","6","7","8","9"]
```

##### 使用行专列函数 explode
``` sql
select explode(split(concat_ws(',', '1', '2', '3', '4', '5', '6', '7', '8', '9'), ',')) from test;
```

数据为:
```
1
2
3
4
5
6
7
8
9
```

##### 使用lateral view 把其他字段显示出来

``` sql
select a.*, b from test a lateral view explode(split(concat_ws(',', '1', '2', '3', '4', '5', '6', '7', '8', '9'), ',')) t as b;
```

数据为：
```
a	aaa	1
a	aaa	2
a	aaa	3
a	aaa	4
a	aaa	5
a	aaa	6
a	aaa	7
a	aaa	8
a	aaa	9
b	bbb	1
b	bbb	2
b	bbb	3
b	bbb	4
b	bbb	5
b	bbb	6
b	bbb	7
b	bbb	8
b	bbb	9
…………
```
