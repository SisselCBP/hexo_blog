---
title: SQL语句怎么写
date: 2019-03-29 02:17:40
tags:
- 淘宝店
- SQL
- 面试
---

持续更新，因为近期写了好多sql语句，所以来列个入门级的内容，掌握了这些再去多刷刷题就好啦

<!-- more -->

## 随便写写

- 一般来说，我们写一个查询从两个方向考虑，一个是看输出的格式，比如直接输出成rows，或者要 create table，或者修改一些输出格式，比如行变列，列变行。
- 这一层面一般是比较简单的，只要得到数据，都可以用 `select a.xxx as 输出列的名字, b.yyy as 输出列的名字` 来封装好，其他的输出无非是format一下
    - `create table as/like`
    - `group_concat()【行变列】`
    - `case xx when xx then xx else xx end` 
- 之后就是拨开输出的层面，去看我们需要什么数据，在哪些库里。主要是查什么？
    - 这几个商品 `Item`、`ID`
- 一般都需要用到分组
    - 每一天的最大值 `group by Date`
    - 每个班级的前三名 `group by ClassID`
- 排序
    - 前三名 `order by score desc`
- 表的连接，根据不同的情况，一般需要
    - `outer join`
    - `inner join`
    - `select xx,xx from a, b where a.x=b.x`
- 注意一些条件的写法
    - `group by having`
    - `where`
    - `xxx join table_a on condition`
- 其他
    - 唯一性 `distinct`
    - 升降序 `desc`、`asc
    - top语句 【不一定支持】
    - like、通配符
    - limit
    - count()
    - 求和，平均值
    - 最大最小值
    - `isnull()`

## 常见的题型

- 分组后的top x【每个班的最高分】
- 计数，有多少个`count()`
- 连续几天签到
- 简单的循环【变量】
- 行列变换
- 连接的种类，left join/right join/outer join等
- null的处理
- ...

还补充点啥

剩下的就是多写多搜了，mysql报错比起其他语言来说难调试很多。。

## 除了select

当然还有

- 建table和view
- 加索引
- 加列、改列
- update、insert
- 不同的清空表的方法
- 这就太多了。。自己搜面试题吧

