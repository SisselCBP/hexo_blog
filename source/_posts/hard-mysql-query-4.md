---
title: 四个比较难的SQL语句
date: 2019-04-04 03:28:04
tags:
- MySQL
- 笔试题
---

前段时间帮一个同学做了套MySQL的笔试题目，感觉有一定难度，有一段时间没写过SQL语句了，练练手，大佬不要嘲笑嗝。

<!-- more -->

![](https://md.byr.moe/uploads/upload_f428d7b7e4c893fa8982e6b8ff24ac43.png)
## Create Table 'Sales'
```mysql
CREATE TABLE `sales` (
  `id` int(11) NOT NULL,
  `Sales_Date` date NOT NULL,
  `Customer_ID` varchar(50) NOT NULL,
  `Item` varchar(50) NOT NULL,
  `Amount` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

ALTER TABLE `sales`
  ADD PRIMARY KEY (`id`),
  ADD UNIQUE KEY `id` (`id`);

ALTER TABLE `sales`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=14;
COMMIT;
```

![](https://md.byr.moe/uploads/upload_e92b631726f33f96c7fd01f47ea7a2a4.png)

## Init Data

```mysql
INSERT INTO `sales` (`id`, `Sales_Date`, `Customer_ID`, `Item`, `Amount`) VALUES
(1, '2018-08-01', 'AAA', 'Apple', 600),
(2, '2018-08-08', 'AAA', 'Nike', 500),
(3, '2018-08-10', 'AAA', 'Kindle', 900),
(4, '2018-07-22', 'BBB', 'Cola', 400),
(5, '2018-09-12', 'BBB', 'LEGO', 240),
(6, '2018-09-13', 'BBB', 'PS4', 2000),
(7, '2018-10-15', 'CCC', 'SWITCH', 2200),
(8, '2018-08-08', 'AAA', 'Adidas', 5000),
(9, '2018-08-09', 'AAA', 'Douban', 50),
(10, '2018-07-23', 'BBB', 'Paper', 800),
(11, '2018-08-13', 'BBB', 'Phone', 851),
(12, '2018-07-12', 'AAA', 'Apple', 5600),
(13, '2018-08-11', 'BBB', 'Cola', 2300);
```

![](https://md.byr.moe/uploads/upload_4dbfb98b1d2fd9618b93474b31e8c6bc.png)

## Query_1

We need to calculate the amount for each user and month, so we will use `group by sales_month, customer_id`. And then we can add a condition to make the total_amount between 2000 and 10000. `Month(Date)` can calculate the month of the date. 

```mysql
SELECT month(Sales_Date) as sales_month, Customer_ID, sum(amount) as total_amount 
FROM `sales` 
GROUP BY 
    customer_id, sales_month 
having 
    total_amount BETWEEN 2000 and 10000;
```

## Query_2

这个好难，感觉leetcode的hard难度都不止。。我可以做到这样列出来，然后

```mysql
SELECT month(Sales_Date) as sales_month, Customer_ID, item
from sales as s1 where 
	(SELECT count(1) from sales as s2 where 
     month(s1.Sales_Date)=month(s2.Sales_Date) 
     and s1.Customer_ID=s2.Customer_ID 
     and s1.amount > s2.Amount) < 3 
order by sales_month asc, Customer_ID, amount desc;
```

![](https://md.byr.moe/uploads/upload_ac6188bcf47029f1bfc299f69332d3ea.png)

```mysql
SELECT month(Sales_Date) as sales_month, Customer_ID, GROUP_CONCAT(item SEPARATOR '|') as top3
from sales as s1 where 
    (SELECT count(1) from sales as s2 where 
     month(s1.Sales_Date)=month(s2.Sales_Date) 
     and s1.Customer_ID=s2.Customer_ID 
     and s1.amount > s2.Amount) < 3 
group by sales_month, Customer_ID
order by sales_month asc, Customer_ID, amount desc
```

![](https://md.byr.moe/uploads/upload_bed54b471e4dd2071f08c9fed9ad2319.png)

First of all, we should know how to make multiple rows to one row, we can use `GROUP_CONCAT`.
Then we need to process the date group by group, we use a sub query and the condition `< 3` .
And I think get the order right is also important. The first is sales_month, then customer_id, and finally amount.

## Query_3

```mysql
with t1 as (
select Item, sum(amount) as total from sales where year(Sales_Date)=2018 and month(Sales_Date)=7 group by Item
), t2 as (
select Item, sum(amount) as total from sales where year(Sales_Date)=2018 and month(Sales_Date)=8 group by Item
), p as ( SELECT t1.item as item, -((case when t2.item is null then 0 else t2.total end)-t1.total) as MoM_Decrease FROM t1
		LEFT JOIN t2 ON t1.item = t2.item
	UNION
	(SELECT t2.item as item, -(t2.total-(case when t1.item is null then 0 else t1.total end)) as MoM_Decrease FROM t1
		RIGHT JOIN t2 ON t1.item = t2.item)
),q as (select * from p 
	where p.MoM_Decrease >= 0
	order by p.MoM_Decrease desc limit 10)
select 
	(@rowNum:=@rowNum+1) Rank_,q.*
from 
	q,(select (@rowNum :=0)) k;
```

Pre-query t1 and t2 query for the purchase log of July and August. And query p is calculate for the MoM_Decrease for each item, we use the FULL OUTER JOIN here. Then we can add the rank for the result.

## Query_4

```mysql
with t as (
select Customer_ID, Sales_Date from sales where year(Sales_Date)=2018 and month(Sales_Date)=8
)
select a.customer_ID
 	from t as a, t as b, t as c
   where a.sales_date+1=b.sales_date and a.sales_date+2=c.sales_date
 	and a.customer_id = b.customer_id and a.customer_id = c.customer_id GROUP BY Customer_ID
```

First we use pre-query to find the rows in 2018/08. After that we can use a simple query to find the customers who have purchased in at least three consecutive.

在mysql的规范中，并不包含连续性的相关方法，所以按照通常来说，我们应该使用其他方式查询该题所需的数据【例如一些脚本语言，可以使用更方便快捷的算法】。




