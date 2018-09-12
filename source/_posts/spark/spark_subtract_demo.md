---
title: spark subtract 算子的应用
tags: [spark,scala]
date: 2017-12-03
categories: Spark
---

#### 问题引入
有两个集合A和B
* A集合存放历史数据
* B集合存放当前计算出的数据

现在需要把存在于历史数据集合中，但不存在当前计算集合中的数据标出，并且同集合B，组成新的集合C

####  相对补集(差集)
若A和B是集合，则A在B中的相对补集是这样一个集合：其元素属于B但不属于A，B - A = { x| x∈B且x∉A}。

#### subtract() 算子

spark 算子 subtract() 正好实现了我们以上需求。

> A.subtract(B) 返回在A中出现, 并且不在B中出现的元素


``` scala

val A_rdd = sc.makeRDD(Seq(1,2,2,3))
val B_rdd = sc.makeRDD(Sel(3, 4))

val AB_diff_rdd = A_rdd.subtract(B_rdd)
val C_rdd = B_rdd.union(AB_diff_rdd)

```
