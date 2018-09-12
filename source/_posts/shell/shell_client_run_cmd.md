---
title: 通过shell脚本实现交互式客户端的输入
tags: [shell,client]
date: 2017-10-21
categories: Shell
---

#### 需求
* 在不进入交互式模式下, 执行交互式下的命令

#### 脚本模板
``` shell
命令 <<EOF
交互语句
EOF
```

#### Mysql使用例子

``` shell
mysql -uroot -hlocalhost -proot <<EOF

create database if not exists test;
use test;

create table if not exists test(
  id int(2),
  name varchar(10)
);

insert into test(id, name) values(1, "aaa");
insert into test(id, name) values(2, "bbb");
insert into test(id, name) values(3, "ccc");
insert into test(id, name) values(4, "ddd");
insert into test(id, name) values(5, "fff");

select id, name from test;

EOF
```
