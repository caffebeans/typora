[TOC]



#### [182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

编写一个 SQL 查询，查找 `Person` 表中所有重复的电子邮箱。

**示例：**

~~~mysql

+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
~~~

根据以上输入，你的查询应返回以下结果：

```
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```



方法一：使用 `GROUP BY` 和临时表

~~~sql
select Email from
(
  select Email, count(Email) as num
  from Person
  group by Email
) as statistic
where num > 1;
~~~

方法二：使用 `GROUP BY` 和 `HAVING` 条件

~~~sql
select Email
from Person
group by Email
having count(Email) > 1;
~~~

方法三：使用 `GROUP BY` 和 `HAVING` 条件

~~~mysql
select distinct a.Email from 
Person as a,
Person as b
where a.Email=b.Email and a.Id<>b.Id;
~~~





#### [183. 从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

某网站包含两个表，`Customers` 表和 `Orders` 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

`Customers` 表：

~~~mysql
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
~~~

`Orders` 表：

~~~mysql
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
~~~

例如给定上述表格，你的查询应返回：

~~~mysql
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
~~~

方法一：使用子查询和 `NOT IN` 子句

~~~ sql

select customers.name as 'Customers'
from customers
where customers.id not in
(
    select customerid from orders
);
~~~

方法二： 使用左连接加`is null` 

~~~mysql
SELECT L1.Name Customers
    FROM Customers L1 LEFT JOIN Orders L2
    ON L1.Id = L2.CustomerId
    WHERE L2.Id IS NULL
~~~

> 在使用左连接时，==null与is null的判断是不同的



#### [1777. 每家商店的产品价格](https://leetcode-cn.com/problems/products-price-for-each-store/)

表：`Products`

~~~mysql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| store       | enum    |
| price       | int     |
+-------------+---------+
(product_id,store) 是这个表的主键。
store 字段是枚举类型，它的取值为以下三种 ('store1', 'store2', 'store3') 。
price 是该商品在这家商店中的价格。
~~~

写出一个 SQL 查询语句，查找每种产品在各个商店中的价格。

可以以 **任何顺序** 输出结果。

查询结果格式如下例所示

~~~mysql
Products 表：
+-------------+--------+-------+
| product_id  | store  | price |
+-------------+--------+-------+
| 0           | store1 | 95    |
| 0           | store3 | 105   |
| 0           | store2 | 100   |
| 1           | store1 | 70    |
| 1           | store3 | 80    |
+-------------+--------+-------+
Result 表：
+-------------+--------+--------+--------+
| product_id  | store1 | store2 | store3 |
+-------------+--------+--------+--------+
| 0           | 95     | 100    | 105    |
| 1           | 70     | null   | 80     |
+-------------+--------+--------+--------+
产品 0 的价格在商店 1 为 95 ，商店 2 为 100 ，商店 3 为 105 。
产品 1 的价格在商店 1 为 70 ，商店 3 的产品 1 价格为 80 ，但在商店 2 中没有销售。
~~~



方法一：使用`group by`+`sum` 

第一种行转列，如果product没有在对应的商店出现，则记为null值。注意这里用group by做聚合时，sum会把每个product的price相加，sum中加入if的作用就是限定对应store的price可以被加，如果不符合条件就输出null，而在sum中如果含有null值则会直接输出Null

~~~ mysql
select
    product_id,
    sum(if(store='store1',price,null)) store1,
    sum(if(store='store2',price,null)) store2,
    sum(if(store='store3',price,null)) store3
from
    Products
group by
    1
~~~



方法二：穷举所有的store记录，用left join联立Products表

~~~ mysql
select
    distinct p.product_id, a.price store1, b.price store2, c.price store3
from
    Products p
left join 
    (select * from Products where store = 'store1') a on p.product_id = a.product_id
left join
    (select * from Products where store = 'store2') b on p.product_id = b.product_id
left join
    (select * from Products where store = 'store3') c on p.product_id = c.product_id
~~~



#### [178. 分数排名](https://leetcode-cn.com/problems/rank-scores/)

编写一个 SQL 查询来实现分数排名。

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。

~~~mysql
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
例如，根据上述给定的 `Scores` 表，你的查询应该返回（按分数从高到低排列）

+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
~~~

> 常用的排名函数

`row_number()` 

Row_number() 在排名是序号 连续 不重复，即使遇到表中的两个一样的数值亦是如此

`rank()` 

Rank() 函数会把要求排序的值相同的归为一组且每组序号一样，排序不会连续执行

`dense_rank()`

Dense_rank() 排序是连续的，也会把相同的值分为一组且每组排序号一样

`ntile()`

Ntile(group_num) 将所有记录分成group_num个组，每组序号一样

方法一： 

~~~ mysql
   select Score,
    DENSE_RANK() OVER (
        ORDER BY Score desc
    ) `rank`
    FROM
    Scores; 

~~~



#### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

`Employee` 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

~~~ mysql
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
| 4  | Sam   | 60000  | 2            |
| 5  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
~~~

`Department` 表包含公司所有部门的信息。

~~~mysql
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
~~~

方法：使用 `JOIN` 和 `IN` 语句

因为 **Employee** 表包含 *Salary* 和 *DepartmentId* 字段，我们可以以此在部门内查询最高工资。

~~~mysql
SELECT
    DepartmentId, MAX(Salary)
FROM
    Employee
GROUP BY DepartmentId;
~~~

>注意：有可能有多个员工同时拥有最高工资，所以最好在这个查询中不包含雇员名字的信息。

~~~ mysql
| DepartmentId | MAX(Salary) |
|--------------|-------------|
| 1            | 90000       |
| 2            | 80000       |
~~~

然后，我们可以把表 **Employee** 和 **Department** 连接，再在这张临时表里用 `IN` 语句查询部门名字和工资的关系

~~~ sql
SELECT
    Department.name AS 'Department',
    Employee.name AS 'Employee',
    Salary
FROM
    Employee
        JOIN
    Department ON Employee.DepartmentId = Department.Id
WHERE
    (Employee.DepartmentId , Salary) IN
    (   SELECT
            DepartmentId, MAX(Salary)
        FROM
            Employee
        GROUP BY DepartmentId
	);
~~~

~~~ sql
| Department | Employee | Salary |
|------------|----------|--------|
| Sales      | Henry    | 80000  |
| IT         | Max      | 90000  |
~~~

class Solution {
    public int maxCount(int m, int n, int[][] ops) {
        for (int[] op : ops) {
            m = Math.min(m, op[0]);
            n = Math.min(n, op[1]);
        }
        return m * n;
    }
}



#### [196. 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

编写一个 SQL 查询，来删除 `Person` 表中所有重复的电子邮箱，重复的邮箱里只保留 **Id** *最小* 的那个。

~~~mysql
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id 是这个表的主键。

+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
~~~

方法一：使用 `DELETE` 和 `WHERE` 子句

~~~ mysql
SELECT p1.*
FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email;
~~~

~~~ mysql
SELECT p1.*
FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id;

~~~

因为我们已经得到了要删除的记录，所以我们最终可以将该语句更改为 `DELETE`

~~~mysql
DELETE p1 FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id
~~~

方法二：使用 开窗函数

~~~ sql
delete 
from Person
where Id in
    (
        select Id
        from
            (
                select Id,
                    row_number() over(partition by Email order by Id) rn
                from Person
            ) t1
        where rn>1
    )
~~~



#### [1149. 文章浏览 II]()

表views 

~~~ mysql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| article_id    | int     |
| author_id     | int     |
| viewer_id     | int     |
| view_date     | date    |
+---------------+---------+
此表无主键，因此可能会存在重复行。此表的每一行都表示某人在某天浏览了某位作者的某篇文章。 请注意，同一人的 author_id 和 viewer_id 是相同的。
~~~

编写一条 SQL 查询来找出在同一天阅读至少两篇文章的人，结果按照 id 升序排序。

查询结果的格式如下：

~~~ mysql
Views table:
+------------+-----------+-----------+------------+
| article_id | author_id | viewer_id | view_date  |
+------------+-----------+-----------+------------+
| 1          | 3         | 5         | 2019-08-01 |
| 3          | 4         | 5         | 2019-08-01 |
| 1          | 3         | 6         | 2019-08-02 |
| 2          | 7         | 7         | 2019-08-01 |
| 2          | 7         | 6         | 2019-08-02 |
| 4          | 7         | 1         | 2019-07-22 |
| 3          | 4         | 4         | 2019-07-21 |
| 3          | 4         | 4         | 2019-07-21 |
+------------+-----------+-----------+------------+

Result table:
+------+
| id   |
+------+
| 5    |
| 6    |
+------+
~~~



方法一：`GROUP BY` 和 `DISTINCT`

**算法**

根据题目一步一步的拆分成子任务：

1. 首先题目要求是**同一天**，所以需要根据时间聚合记录，使用 `GROUP BY` 聚合。

~~~ mysql
GROUP BY view_date
~~~

2. 其次是至少阅读两篇文章的人。通过这句话我们可以知道还需要根据人来聚合，计算每个人阅读的文章数。在 GROUP BY 的基础上使用 HAVING 过滤条件。因为表中可能有重复的数据，所以还要对 article_id 做去重处理。

~~~ mysql
GROUP BY viewer_id
HAVING COUNT(DISTINCT article_id) >= 2
~~~

3. 然后将**结果按照 `id` 升序排序**，这个只需要使用 `ORDER BY` 对结果进行排序。
4. 最后将上面三步联合起来就是我们需要的数据。但是结果依然有可能重复，所以需要再对结果去重。

~~~ mysql
SELECT DISTINCT viewer_id AS id
FROM Views
GROUP BY view_date, viewer_id
HAVING COUNT(DISTINCT article_id) >= 2
ORDER BY viewer_id
~~~



#### [1294. 不同国家的天气类型](https://leetcode-cn.com/problems/weather-type-in-each-country/)

思路和心得：

1.分组

2.统计

3.判断

~~~ mysql
# Write your MySQL query statement below
SELECT country_name AS 'country_name',
    CASE 
        WHEN AVG(w1.weather_state) <= 15 THEN 'Cold' 
        WHEN AVG(w1.weather_state) >= 25 THEN 'Hot'
        ELSE 'Warm'
    END
        AS 'weather_type'
FROM 
    Countries AS c1
    INNER JOIN Weather AS w1
    ON c1.country_id = w1.country_id
WHERE w1.day BETWEEN '2019-11-01' AND '2019-11-30'
GROUP BY c1.country_id;

~~~



~~~mysql
# Write your MySQL query statement below
SELECT country_name AS 'country_name',
    CASE 
        WHEN AVG(w1.weather_state) <= 15 THEN 'Cold' 
        WHEN AVG(w1.weather_state) >= 25 THEN 'Hot'
        ELSE 'Warm'
    END
        AS 'weather_type'
FROM 
    Countries AS c1
    INNER JOIN Weather AS w1
    ON c1.country_id = w1.country_id
WHERE YEAR(w1.day) = 2019 AND MONTH(w1.day) = 11
GROUP BY c1.country_id
;
~~~



~~~mysql
# Write your MySQL query statement below
SELECT country_name AS 'country_name',
    CASE 
        WHEN AVG(w1.weather_state) <= 15 THEN 'Cold' 
        WHEN AVG(w1.weather_state) >= 25 THEN 'Hot'
        ELSE 'Warm'
    END
        AS 'weather_type'
FROM 
    Countries AS c1
    INNER JOIN Weather AS w1
    ON c1.country_id = w1.country_id
WHERE DATE_FORMAT(w1.day, '%Y-%m') = '2019-11'
GROUP BY c1.country_id
;
~~~



#### [1126. 查询活跃业务](https://leetcode-cn.com/problems/active-businesses/)

事件表：`Events`

~~~ mysql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| business_id   | int     |
| event_type    | varchar |
| occurences    | int     | 
+---------------+---------+
~~~

写一段 SQL 来查询所有活跃的业务。

如果一个业务的某个事件类型的发生次数大于此事件类型在所有业务中的平均发生次数，并且该业务至少有两个这样的事件类型，那么该业务就可被看做是活跃业务。

查询结果格式如下所示：

~~~mysql
Events table:
+-------------+------------+------------+
| business_id | event_type | occurences |
+-------------+------------+------------+
| 1           | reviews    | 7          |
| 3           | reviews    | 3          |
| 1           | ads        | 11         |
| 2           | ads        | 7          |
| 3           | ads        | 6          |
| 1           | page views | 3          |
| 2           | page views | 12         |
+-------------+------------+------------+

结果表
+-------------+
| business_id |
+-------------+
| 1           |
+-------------+ 
'reviews'、 'ads' 和 'page views' 的总平均发生次数分别是 (7+3)/2=5, (11+7+6)/3=8, (3+12)/2=7.5。
id 为 1 的业务有 7 个 'reviews' 事件（大于 5）和 11 个 'ads' 事件（大于 8），所以它是活跃业务。
~~~

方法一：`JOIN` 和 `GROUP BY`

思路
首先根据题目，必须要知道所有业务中的平均发生次数。所以需要计算每个业务的平均值，可以使用 AVG 函数和 GROUP BY 计算每个 event_type 的平均值。

~~~mysql
SELECT event_type, AVG(occurences) AS eventAvg
FROM Events
GROUP BY event_type
~~~

拿到平均值后使用 `JOIN` 将新的数据和老数据根据 `event_type` 联合在一起。判断老数据的 `occurences` 是否大于平均值。

```mysql
SELECT business_id
FROM Events AS e
JOIN (
    SELECT event_type, AVG(occurences) AS eventAvg
    FROM Events
    GROUP BY event_type
) AS e1 
ON e.event_type = e1.event_type
WHERE e.occurences > e1.eventAvg
```

题目最后还要求**至少有两个这样的事件类型**，所以需要再用一次 `GROUP BY` 将每一个业务聚合并判断事件类型的数量。

~~~ mysql
SELECT business_id
FROM Events AS e
JOIN (
    SELECT event_type, AVG(occurences) AS eventAvg
    FROM Events
    GROUP BY event_type
) AS e1 
ON e.event_type = e1.event_type
WHERE e.occurences > e1.eventAvg
GROUP BY business_id
HAVING COUNT(*) >= 2
~~~



#### [1107. 每日新用户统计](https://leetcode-cn.com/problems/new-users-daily-count/)

`Traffic` 表：

~~~ mysql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| activity      | enum    |
| activity_date | date    |
+---------------+---------+
~~~



编写一个 SQL 查询，以查询从今天起最多 90 天内，每个日期该日期首次登录的用户数。假设今天是 **2019-06-30**.

查询结果格式如下例所示：

~~~ mysql
Traffic 表：
+---------+----------+---------------+
| user_id | activity | activity_date |
+---------+----------+---------------+
| 1       | login    | 2019-05-01    |
| 1       | homepage | 2019-05-01    |
| 1       | logout   | 2019-05-01    |
| 2       | login    | 2019-06-21    |
| 2       | logout   | 2019-06-21    |
| 3       | login    | 2019-01-01    |
| 3       | jobs     | 2019-01-01    |
| 3       | logout   | 2019-01-01    |
| 4       | login    | 2019-06-21    |
| 4       | groups   | 2019-06-21    |
| 4       | logout   | 2019-06-21    |
| 5       | login    | 2019-03-01    |
| 5       | logout   | 2019-03-01    |
| 5       | login    | 2019-06-21    |
| 5       | logout   | 2019-06-21    |
+---------+----------+---------------+

Result 表：
+------------+-------------+
| login_date | user_count  |
+------------+-------------+
| 2019-05-01 | 1           |
| 2019-06-21 | 2           |
+------------+-------------+
请注意，我们只关心用户数非零的日期.
ID 为 5 的用户第一次登陆于 2019-03-01，因此他不算在 2019-06-21 的的统计内。
~~~

方法一：

~~~ mysql
select dt as login_date, count(*) as user_count
from(
    select user_id,min(activity_date) as dt
    from traffic
    where activity = 'login'
    group by user_id
    having min(activity_date) >= date_sub('2019-06-30',interval 90 day) ) as t
group by dt
~~~



方法二：

~~~mysql
-- 首先求出每个用户首次登陆的时间，然后把登录时间在4-30号之前的id丢掉，然后按照登录时间分组，统计登陆的人数
select login_date,count(user_id) user_count
from (select user_id, min(activity_date) login_date from Traffic
where activity='login'
group by user_id) t
where datediff('2019-06-30',login_date)<=90
group by login_date;
~~~



#### [1517. 查找拥有有效邮箱的用户](https://leetcode-cn.com/problems/find-users-with-valid-e-mails/)

解题思路

**REGEXP** 就是 regular expression 正则表达式 的意思

^  表示以后面的字符为开头
[] 表示括号内任意字符

\- 表示连续

\*  表示重复前面任意字符任意次数
\ 用来转义后面的特殊字符，以表示字符原本的样子，而不是将其作为特殊字符使用
$ 表示以前面的字符为结尾

前缀名以字母开头：^[a-zA-Z]
前缀名包含字母（大写或小写）、数字、下划线_、句点. 和 或 横杠-：[a-zA-Z0-9\_\.\-]*
以域名'@leetcode.com'结尾：@leetcode\.com$

~~~mysql
# Write your MySQL query statement below

select * from users
where mail REGEXP '^[a-zA-Z][a-zA-Z0-9\_\.\-]*@leetcode\.com$'
~~~

#### [1251. 平均售价](https://leetcode-cn.com/problems/average-selling-price/)

Table: `Prices`

~~~
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| start_date    | date    |
| end_date      | date    |
| price         | int     |
+---------------+---------+
(product_id，start_date，end_date) 是 Prices 表的主键。
Prices 表的每一行表示的是某个产品在一段时期内的价格。
每个产品的对应时间段是不会重叠的，这也意味着同一个产品的价格时段不会出现交叉。
~~~

~~~mysql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| purchase_date | date    |
| units         | int     |
+---------------+---------+
UnitsSold 表没有主键，它可能包含重复项。
UnitsSold 表的每一行表示的是每种产品的出售日期，单位和产品 id。
~~~

~~~mysql
Prices table:
+------------+------------+------------+--------+
| product_id | start_date | end_date   | price  |
+------------+------------+------------+--------+
| 1          | 2019-02-17 | 2019-02-28 | 5      |
| 1          | 2019-03-01 | 2019-03-22 | 20     |
| 2          | 2019-02-01 | 2019-02-20 | 15     |
| 2          | 2019-02-21 | 2019-03-31 | 30     |
+------------+------------+------------+--------+
 
UnitsSold table:
+------------+---------------+-------+
| product_id | purchase_date | units |
+------------+---------------+-------+
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |
+------------+---------------+-------+

Result table:
+------------+---------------+
| product_id | average_price |
+------------+---------------+
| 1          | 6.96          |
| 2          | 16.96         |
+------------+---------------+
平均售价 = 产品总价 / 销售的产品数量。
产品 1 的平均售价 = ((100 * 5)+(15 * 20) )/ 115 = 6.96
产品 2 的平均售价 = ((200 * 15)+(30 * 30) )/ 230 = 16.96
~~~

方法一：`JOIN`

本题需要计算每个产品的平均售价，平均售价 = 销售总额 / 总数量，因此我们只需要计算除每个产品的销售总额和总数量即可。

总数量可以直接使用 UnitsSold 计算得出，使用 GROUP BY 和 SUM 函数即可：

~~~ mysql
SELECT product_id, SUM(units) FROM UnitsSold GROUP BY product_id
~~~



​      因为每个产品不同时期的售价不同，因此在计算销售总额之前要先分别计算每个价格的销售总额。每个价格的销售总额为 对应时间内的价格 * 对应时间内的数量对应时间内的价格∗对应时间内的数量。因为价格和时间在 Prices 表中，数量在 UnitsSold 表中，这两个表通过 product_id 关联，那么我们就可以使用 JOIN 将两个表连接，通过 WHERE 查询对应时间内每个产品的价格和数量，并计算出对应的销售总额。

~~~mysql
SELECT
    Prices.product_id AS product_id,
    Prices.price * UnitsSold.units AS sales,
    UnitsSold.units AS units
FROM Prices 
JOIN UnitsSold ON Prices.product_id = UnitsSold.product_id
WHERE UnitsSold.purchase_date BETWEEN Prices.start_date AND Prices.end_date
~~~

 计算出产品每个价格的销售总额后，同样的使用 `SUM` 函数计算出产品所有时间的销售总额，然后除以总数量并使用 `ROUND` 函数保留两位小数即可。

~~~mysql
SELECT
    product_id,
    Round(SUM(sales) / SUM(units), 2) AS average_price
FROM (
    SELECT
        Prices.product_id AS product_id,
        Prices.price * UnitsSold.units AS sales,
        UnitsSold.units AS units
    FROM Prices 
    JOIN UnitsSold ON Prices.product_id = UnitsSold.product_id
    WHERE UnitsSold.purchase_date BETWEEN Prices.start_date AND Prices.end_date
) T
GROUP BY product_id
~~~



#### [570. 至少有5名直接下属的经理](https://leetcode-cn.com/problems/managers-with-at-least-5-direct-reports/)

`Employee` 表包含所有员工和他们的经理。每个员工都有一个 Id，并且还有一列是经理的 Id。

~~~sql
+------+----------+-----------+----------+
|Id    |Name 	  |Department |ManagerId |
+------+----------+-----------+----------+
|101   |John 	  |A 	      |null        |
|102   |Dan 	  |A 	      |101         |
|103   |James 	  |A 	    |101       |
|104   |Amy 	  |A 	      |101         |
|105   |Anne 	  |A 	      |101         |
|106   |Ron 	  |B 	      |101         |
+------+----------+-----------+----------+
~~~

给定 `Employee` 表，请编写一个SQL查询来查找至少有5名直接下属的经理。对于上表，您的SQL查询应该返回：

```mysql
+-------+
| Name  |
+-------+
| John  |
+-------+
```

> 没有人是自己的下属

方法：使用**临时表**进行连接

~~~sql

SELECT
    Name
FROM
    Employee AS t1 JOIN
    (SELECT
        ManagerId
    FROM
        Employee
    GROUP BY ManagerId
    HAVING COUNT(ManagerId) >= 5) AS t2
    ON t1.Id = t2.ManagerId;
~~~



#### [1098. 小众书籍](https://leetcode-cn.com/problems/unpopular-books/)

书籍表 `Books`：

~~~sql
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| book_id        | int     |
| name           | varchar |
| available_from | date    |
+----------------+---------+
book_id 是这个表的主键。
~~~

订单表 `Orders`

~~~sql
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| order_id       | int     |
| book_id        | int     |
| quantity       | int     |
| dispatch_date  | date    |
+----------------+---------+
order_id 是这个表的主键。
book_id  是 Books 表的外键。
~~~

你需要写一段 SQL 命令，筛选出过去一年中订单总量 少于10本 的 书籍 。

注意：不考虑 上架（available from）距今 不满一个月 的书籍。并且 假设今天是 2019-06-23 。

~~~sql
Books 表：
+---------+--------------------+----------------+
| book_id | name               | available_from |
+---------+--------------------+----------------+
| 1       | "Kalila And Demna" | 2010-01-01     |
| 2       | "28 Letters"       | 2012-05-12     |
| 3       | "The Hobbit"       | 2019-06-10     |
| 4       | "13 Reasons Why"   | 2019-06-01     |
| 5       | "The Hunger Games" | 2008-09-21     |
+---------+--------------------+----------------+

Orders 表：
+----------+---------+----------+---------------+
| order_id | book_id | quantity | dispatch_date |
+----------+---------+----------+---------------+
| 1        | 1       | 2        | 2018-07-26    |
| 2        | 1       | 1        | 2018-11-05    |
| 3        | 3       | 8        | 2019-06-11    |
| 4        | 4       | 6        | 2019-06-05    |
| 5        | 4       | 5        | 2019-06-20    |
| 6        | 5       | 9        | 2009-02-02    |
| 7        | 5       | 8        | 2010-04-13    |
+----------+---------+----------+---------------+

Result 表：
+-----------+--------------------+
| book_id   | name               |
+-----------+--------------------+
| 1         | "Kalila And Demna" |
| 2         | "28 Letters"       |
| 5         | "The Hunger Games" |
+-----------+--------------------+
~~~

解法一：

~~~sql
SELECT b.book_id, b.name
FROM (
    SELECT book_id, name
    FROM books
    WHERE DATEDIFF('2019-06-23',available_from) >= 30
) b LEFT JOIN (
    SELECT book_id, quantity
    FROM orders
    WHERE DATEDIFF('2019-06-23',dispatch_date) <= 365
) o ON b.book_id=o.book_id
GROUP BY b.book_id
HAVING SUM(IFNULL(quantity,0))<10
~~~

解法二：

- 考察的知识点主要是left join的使用以及on和where的辨析
	注意两个条件的区别：

- 过去一年内订单总量少于10本：因为有的书根本没有订单，所以该条件应该让left join对应的on去筛选，没有订单的至少返回null，后续可以通过ifnull()把null转换成0
	不考虑上架不满一个月的书：应该直接通过where筛除，不符合条件的连null都不应该有，否则会混入结果中导致错误
- 本题的时间边界条件比较模糊，“一个月”、“一年”都没有比较清楚的描述，用例也没那么强，大致对即可通过

~~~sql
select b.book_id, name
from books b left join orders o
on b.book_id = o.book_id and dispatch_date >= '2018-06-23'
where available_from < '2019-05-23'
group by b.book_id
having ifnull(sum(quantity), 0) < 10

~~~



#### [1193. 每月交易 I](https://leetcode-cn.com/problems/monthly-transactions-i/)

Table: `Transactions`

~~~sql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| country       | varchar |
| state         | enum    |
| amount        | int     |
| trans_date    | date    |
+---------------+---------+
id 是这个表的主键。
该表包含有关传入事务的信息。
state 列类型为 “[”批准“，”拒绝“] 之一。
~~~

编写一个 sql 查询来查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额。

~~~sql
Transactions table:
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |
+------+---------+----------+--------+------------+

Result table:
+----------+---------+-------------+----------------+--------------------+-----------------------+
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
+----------+---------+-------------+----------------+--------------------+-----------------------+
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |
+----------+---------+-------------+----------------+--------------------+-----------------------+

~~~

**预备知识**
本题使用到的 MySQL 函数的说明：

DATE_FORMAT(date, format) ：用于以不同的格式显示日期/时间数据。date 参数是合法的日期，format 规定日期/时间的输出格式。
方法一：`DATE_FORMAT()` 函数、`GROUP BY`

本题要求 **查找每个月和每个国家/地区的事务数及其总金额、已批准的事务数及其总金额**，我们可以将这句话拆分成几个子任务：

1. 查找每个月和每个国家/地区。
	数据表中的 trans_date 是精确到日，我们可以使用 DATE_FORMAT() 函数将日期按照年月 %Y-%m 输出。比如将 2019-01-02 转换成 2019-01 。

> DATE_FORMAT(trans_date, '%Y-%m')

获取到所有的月份后，使用 `GROUP BY` 聚合每个月和每个国家的记录就完成了第一步。

2. 查找总的事务数。

	第一步已经将数据按月和国家聚合，只需要使用 `COUNT` 函数就能获取到总的事务数。

	`COUNT(*) AS trans_count`

3. 查找总金额。

	使用 `SUM` 函数计算总金额。

	`SUM(amount) AS trans_total_amount`

4. 查找已批准的事物数。

5. 已批准的事物的 state 标记为 approved。首先使用 IF 函数将 state = 'approved' 的记录标记为 1，否则为 NULL。再使用 COUNT 计算总量。

	`COUNT(IF(state = 'approved', 1, NULL)) AS approved_count`

	

```sql

SELECT DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(*) AS trans_count,
    COUNT(IF(state = 'approved', 1, NULL)) AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(IF(state = 'approved', amount, 0)) AS approved_total_amount
FROM Transactions
GROUP BY month, country

```



#### [1077. 项目员工 III](https://leetcode-cn.com/problems/project-employees-iii/)

项目表 `Project`：

~~~sql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| project_id  | int     |
| employee_id | int     |
+-------------+---------+
(project_id, employee_id) 是这个表的主键
employee_id 是员工表 Employee 的外键
~~~

员工表 `Employee`：

~~~sql
+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| employee_id      | int     |
| name             | varchar |
| experience_years | int     |
+------------------+---------+
employee_id 是这个表的主键
~~~

写 一个 SQL 查询语句，报告在每一个项目中经验最丰富的雇员是谁。如果出现经验年数相同的情况，请报告所有具有最大经验年数的员工。

查询结果格式在以下示例中：

~~~sql
Project 表：
+-------------+-------------+
| project_id  | employee_id |
+-------------+-------------+
| 1           | 1           |
| 1           | 2           |
| 1           | 3           |
| 2           | 1           |
| 2           | 4           |
+-------------+-------------+

Employee 表：
+-------------+--------+------------------+
| employee_id | name   | experience_years |
+-------------+--------+------------------+
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 3                |
| 4           | Doe    | 2                |
+-------------+--------+------------------+

Result 表：
+-------------+---------------+
| project_id  | employee_id   |
+-------------+---------------+
| 1           | 1             |
| 1           | 3             |
| 2           | 1             |
+-------------+---------------+
employee_id 为 1 和 3 的员工在 project_id 为 1 的项目中拥有最丰富的经验。在 project_id 为 2 的项目中，employee_id 为 1 的员工拥有最丰富的经验。
~~~

`窗口函数方法`

~~~sql
-- 步骤1：先使用窗口函数获取每个project_id下experience_years的排序
select *,
dense_rank() over(partition by p.project_id order by e.experience_years desc) as rank_year
from project p join employee e
using(employee_id)

-- 步骤2：找到排序第一名（或并列）的employee id
select temp.project_id,temp.employee_id
from 
(select *,
dense_rank() over(partition by p.project_id order by e.experience_years desc) as rank_year
from project p join employee e
using(employee_id)) as temp 
where temp.rank_year=1
~~~

`join`

~~~sql
select Project.project_id,Project.employee_id from
Project left join
Employee using(employee_id)
where (Project.project_id,Employee.experience_years) in (
select t1.project_id,max(t2.experience_years) from Project as t1
left join Employee as t2
on t1.employee_id = t2.employee_id
group by project_id)
~~~



#### [1264. 页面推荐](https://leetcode-cn.com/problems/page-recommendations/)

朋友关系列表： `Friendship`

```sql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user1_id      | int     |
| user2_id      | int     |
+---------------+---------+
这张表的主键是 (user1_id, user2_id)。
这张表的每一行代表着 user1_id 和 user2_id 之间存在着朋友关系。
```

喜欢列表： `Likes`

~~~sql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user_id     | int     |
| page_id     | int     |
+-------------+---------+
这张表的主键是 (user_id, page_id)。
这张表的每一行代表着 user_id 喜欢 page_id。
~~~

写一段 SQL  向user_id = 1 的用户，推荐其朋友们喜欢的页面。不要推荐该用户已经喜欢的页面。

你返回的结果中不应当包含重复项。

返回结果的格式如下例所示：

~~~sql
Friendship table:
+----------+----------+
| user1_id | user2_id |
+----------+----------+
| 1        | 2        |
| 1        | 3        |
| 1        | 4        |
| 2        | 3        |
| 2        | 4        |
| 2        | 5        |
| 6        | 1        |
+----------+----------+
 
Likes table:
+---------+---------+
| user_id | page_id |
+---------+---------+
| 1       | 88      |
| 2       | 23      |
| 3       | 24      |
| 4       | 56      |
| 5       | 11      |
| 6       | 33      |
| 2       | 77      |
| 3       | 77      |
| 6       | 88      |
+---------+---------+

Result table:
+------------------+
| recommended_page |
+------------------+
| 23               |
| 24               |
| 56               |
| 33               |
| 77               |
+------------------+
用户1 同 用户2, 3, 4, 6 是朋友关系。
推荐页面为： 页面23 来自于 用户2, 页面24 来自于 用户3, 页面56 来自于 用户3 以及 页面33 来自于 用户6。
页面77 同时被 用户2 和 用户3 推荐。
页面88 没有被推荐，因为 用户1 已经喜欢了它。
~~~

方法一： `UNION ALL`

思路

本题最直观的思路就是找到所有 user_id = 1 的朋友，找到他们喜欢的 page_id，再去掉 user_id = 1 喜欢的 page_id。

~~~sql
SELECT page_id 
FROM Likes WHERE
user_id IN (friends) AND page_id NOT IN (user_id = 1 like page)
~~~

根据上面的伪代码，我们只需要求出 friends 和 user_id = 1 like page 就能完成本题。

首先我们求 friends。通过 Friendship 可以得到所有的朋友。user_id = 1 有可能在 user1_id，也有可能在 user2_id。因此我们需要两个 sql 分别求出朋友。

1. SELECT user1_id AS user_id FROM Friendship WHERE user2_id = 1
	2. SELECT user2_id AS user_id FROM Friendship WHERE user1_id = 1
		使用 UNION ALL 或者 UNION 得到所有的朋友 （UNION ALL 和 UNION 的区别在于前者会去掉重复的行，后者不会，这里使用前者更高效）。

然后我们再求 user_id = 1 like page。这个也很简单，直接对表 Likes 使用 WHERE 语句即可。

~~~sql
SELECT page_id FROM Likes WHERE user_id = 1
~~~

**代码**

~~~sql

SELECT DISTINCT page_id AS recommended_page
FROM Likes
WHERE user_id IN (
    SELECT user1_id AS user_id FROM Friendship WHERE user2_id = 1
    UNION ALL
    SELECT user2_id AS user_id FROM Friendship WHERE user1_id = 1
) AND page_id NOT IN (
    SELECT page_id FROM Likes WHERE user_id = 1
)
~~~

方法二： `CASE WHEN`

~~~sql
SELECT DISTINCT page_id AS recommended_page
FROM Likes
WHERE user_id IN (
    SELECT (
        CASE
        WHEN user1_id = 1 then user2_id
        WHEN user2_id = 1 then user1_id
        END
    ) AS user_id
    FROM Friendship
    WHERE user1_id = 1 OR user2_id = 1
)  AND page_id NOT IN (
    SELECT page_id FROM Likes WHERE user_id = 1
)
~~~



#### [1321. 餐馆营业额变化增长](https://leetcode-cn.com/problems/restaurant-growth/)

Customer

~~~sql
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| name          | varchar |
| visited_on    | date    |
| amount        | int     |
+---------------+---------+
(customer_id, visited_on) 是该表的主键
该表包含一家餐馆的顾客交易数据
visited_on 表示 (customer_id) 的顾客在 visited_on 那天访问了餐馆
amount 是一个顾客某一天的消费总额
~~~

你是餐馆的老板，现在你想分析一下可能的营业额变化增长（每天至少有一位顾客）

写一条 SQL 查询计算以 7 天（某日期 + 该日期前的 6 天）为一个时间段的顾客消费平均值

查询结果格式的例子如下：

查询结果按 visited_on 排序
average_amount 要 保留两位小数，日期数据的格式为 ('YYYY-MM-DD')

~~~sql
Customer 表:
+-------------+--------------+--------------+-------------+
| customer_id | name         | visited_on   | amount      |
+-------------+--------------+--------------+-------------+
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 
+-------------+--------------+--------------+-------------+

结果表:
+--------------+--------------+----------------+
| visited_on   | amount       | average_amount |
+--------------+--------------+----------------+
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |
+--------------+--------------+----------------+

第一个七天消费平均值从 2019-01-01 到 2019-01-07 是 (100 + 110 + 120 + 130 + 110 + 140 + 150)/7 = 122.86
第二个七天消费平均值从 2019-01-02 到 2019-01-08 是 (110 + 120 + 130 + 110 + 140 + 150 + 80)/7 = 120
第三个七天消费平均值从 2019-01-03 到 2019-01-09 是 (120 + 130 + 110 + 140 + 150 + 80 + 110)/7 = 120
第四个七天消费平均值从 2019-01-04 到 2019-01-10 是 (130 + 110 + 140 + 150 + 80 + 110 + 130 + 150)/7 = 142.86
~~~

`讲解并改进评论区大佬用到的窗口函数法`

~~~sql
[你要的操作] OVER ( PARTITION BY  <用于分组的列名>
                    ORDER BY <按序叠加的列名> 
                    ROWS <窗口滑动的数据范围> )
~~~

当前行 - current row
之前的行 - preceding
之后的行 - following
无界限 - unbounded
表示从前面的起点 - unbounded preceding
表示到后面的终点 - unbounded following

> **<窗口滑动的数据范围> 用来限定[ 你要的操作] 所运用的数据的范围，具体有如下这些：**

~~~sql
举例理解一下：
取当前行和前五行：ROWS between 5 preceding and current row --共6行
取当前行和后五行：ROWS between current row and 5 following --共6行
取前五行和后五行：ROWS between 5 preceding and 5 folowing --共11行
~~~

本题中，要的是 按日期的， 金额的累计， 从当天算起共7天
则可理解成 我要的操作是‘累计金额’， 按序叠加的列是‘日期’， 窗口内的数据要‘取当前行和前6行’

不过即使前边的数据不够，窗口函数也会将范围内的数据框住并计算，因此需要最后手动地只要能够完整框住7天*的情况 【绊子1】
另外比较阴损的是，本题的数据中存在着同一用户在某日多次消费的情况，这样一来即使窗口照旧向前取6天就无法覆盖被挤出去的数据了，因此，需要构建一个小表格用来存放每天的金额总量 【绊子2】

~~~sql

SELECT DISTINCT visited_on,
       sum_amount AS amount, 
       ROUND(sum_amount/7, 2) AS average_amount
-- 以上是破解【绊子1】并计算平均值，少用一次窗口函数提高运行速度
FROM (
    SELECT visited_on, 
       SUM(amount) OVER ( ORDER BY visited_on ROWS 6 PRECEDING ) AS sum_amount
    -- 以下是计算每天的金额总量，破解【绊子2】
    FROM (
        SELECT visited_on, 
            SUM(amount) AS amount
        FROM Customer
        GROUP BY visited_on
         ) TT
     ) LL
-- 最后手动只要覆盖完整7天的数据，破解【绊子1】
WHERE DATEDIFF(visited_on, (SELECT MIN(visited_on) FROM Customer)) >= 6
~~~

#### [1045. 买下所有产品的客户](https://leetcode-cn.com/problems/customers-who-bought-all-products/)

`Customer` 表：

~~~sql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| customer_id | int     |
| product_key | int     |
+-------------+---------+
product_key 是 Customer 表的外键。
~~~

`Product` 表：

~~~sql
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_key | int     |
+-------------+---------+
product_key 是这张表的主键。
~~~

写一条 SQL 查询语句，从 `Customer` 表中查询购买了 `Product` 表中所有产品的客户的 id。

**示例：**

~~~sql
Customer 表：
+-------------+-------------+
| customer_id | product_key |
+-------------+-------------+
| 1           | 5           |
| 2           | 6           |
| 3           | 5           |
| 3           | 6           |
| 1           | 6           |
+-------------+-------------+

Product 表：
+-------------+
| product_key |
+-------------+
| 5           |
| 6           |
+-------------+

Result 表：
+-------------+
| customer_id |
+-------------+
| 1           |
| 3           |
+-------------+
购买了所有产品（5 和 6）的客户的 id 是 1 和 3 。
~~~

`解法`

~~~sql
# Write your MySQL query statement below
select customer_id
from Customer
group by customer_id
having count(distinct product_key) in 
(select count(distinct product_key) from Product)
~~~

#### [569. 员工薪水中位数](https://leetcode-cn.com/problems/median-employee-salary/)

`Employee` 表包含所有员工。`Employee` 表有三列：员工Id，公司名和薪水。

~~~
+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|1    | A          | 2341   |
|2    | A          | 341    |
|3    | A          | 15     |
|4    | A          | 15314  |
|5    | A          | 451    |
|6    | A          | 513    |
|7    | B          | 15     |
|8    | B          | 13     |
|9    | B          | 1154   |
|10   | B          | 1345   |
|11   | B          | 1221   |
|12   | B          | 234    |
|13   | C          | 2345   |
|14   | C          | 2645   |
|15   | C          | 2645   |
|16   | C          | 2652   |
|17   | C          | 65     |
+-----+------------+--------+
~~~

请编写SQL查询来查找每个公司的薪水中位数。挑战点：你是否可以在不使用任何内置的SQL函数的情况下解决此问题。

~~~sql
+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|5    | A          | 451    |
|6    | A          | 513    |
|12   | B          | 234    |
|9    | B          | 1154   |
|14   | C          | 2645   |
+-----+------------+--------+
~~~



方法一：`窗口函数`

~~~sql
select Id,Company,Salary
from
(
select Id,Company,Salary, 
row_number()over(partition by Company order by Salary)as ranking,
count(Id) over(partition by Company)as cnt
from Employee
)a
where ranking>=cnt/2 and ranking<=cnt/2+1
~~~

方法二： 排序后再找 *中位数* 【通过】

思路

如果记录本身就是根据 salary 来排名的，那么很容易就能找到 中位数。但 MySQL 本身是没有内置的排名方法的，所以这里会有一些东西需要处理。

算法

根据 salary 排序记录，利用会话变量计算排名。由于不需要级联表，这个方法要比方法一更高效。

~~~sql
SELECT 
    Id, Company, Salary
FROM
    (SELECT 
        e.Id,
        e.Salary,
        e.Company,
        IF(@prev = e.Company, @Rank:=@Rank + 1, @Rank:=1) AS rank,
        @prev:=e.Company
    FROM
        Employee e, (SELECT @Rank:=0, @prev:=0) AS temp
    ORDER BY e.Company , e.Salary , e.Id) Ranking
        INNER JOIN
    (SELECT 
        COUNT(*) AS totalcount, Company AS name
    FROM
        Employee e2
    GROUP BY e2.Company) companycount ON companycount.name = Ranking.Company
WHERE
    Rank = FLOOR((totalcount + 1) / 2)
        OR Rank = FLOOR((totalcount + 2) / 2)
;
~~~

#### [571. 给定数字的频率查询中位数](https://leetcode-cn.com/problems/find-median-given-frequency-of-numbers/)

`Numbers` 表保存数字的值及其频率。

~~~sql
+----------+-------------+
|  Number  |  Frequency  |
+----------+-------------|
|  0       |  7          |
|  1       |  1          |
|  2       |  3          |
|  3       |  1          |
+----------+-------------+
~~~

在此表中，数字为 `0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3`，所以中位数是 `(0 + 0) / 2 = 0`。

~~~sql
+--------+
| median |
+--------|
| 0.0000 |
+--------+
~~~

解题思路
使用sum over(order by ) 对数字个数进行正序和逆序累计，
当某一数字的 正序和逆序累计 均大于 整个序列的数字个数的一半 时即为中位数，
将最后选定的一个或两个中位数进行求均值即可。

![1606296718-IJZeTW-image](https://pic.leetcode-cn.com/1606296718-IJZeTW-image.png)

~~~sql
# Write your MySQL query statement below
select avg(number) median
from
    (select number,
        sum(frequency) over(order by number) asc_accumu,
        sum(frequency) over(order by number desc) desc_accumu
        from numbers) t1, 
    (select sum(frequency) total from numbers) t2
where asc_accumu >= total/2 and desc_accumu >=total/2
~~~

> MY SQL 不用变量，理解简单

如果 n1.Number 为中位数，n1.Number（包含本身）前累计的数字应大于等于总数/2 同时n1.Number（不包含本身）前累计数字应小于等于总数/2

例如：0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3 共12个数

中位数0（包含本身）前累计的数字 7 >=6  0（不包含本身）前累计数字 0 <=6

 中位数0（包含本身）前累计的数字 3 >=3  0（不包含本身）前累计数字 0 <=3
                 
 中位数3（包含本身）前累计的数字 6 >=3  3（不包含本身）前累计数字 3 <=3

~~~~sql
SELECT 
AVG(Number)median 
FROM
(SELECT n1.Number FROM Numbers n1 JOIN Numbers n2 ON n1.Number>=n2.Number 
 GROUP BY 
 n1.Number 
 HAVING 
 SUM(n2.Frequency)>=(SELECT SUM(Frequency) FROM Numbers)/2 
 AND 
 SUM(n2.Frequency)-AVG(n1.Frequency)<=(SELECT SUM(Frequency) FROM Numbers)/2
)s
~~~~

> 低效率代码

~~~sql
# Write your MySQL query statement below
# select sum(frequency)%2 from Numbers
# select Frequency from Numbers a order by a.Number limit 0,1
# avg(tmp.Number) as median
select avg(tmp.number) as median from (
    select 
        n1.Number,
        (select sum(n2.Frequency) from Numbers n2 where n2.Number <= n1.Number order by n2.Number) as asc_f,
        (select sum(n3.Frequency) from Numbers n3 where n3.Number >= n1.Number order by n3.Number) as desc_f
    from 
    Numbers n1
    order by n1.Number
) tmp
where tmp.asc_f >= (select sum(frequency) from numbers)/2 and tmp.desc_f >= (select sum(frequency) from numbers)/2
~~~

#### [615. 平均工资：部门与公司比较](https://leetcode-cn.com/problems/average-salary-departments-vs-company/)

给如下两个表，写一个查询语句，求出在每一个工资发放日，每个部门的平均工资与公司的平均工资的比较结果 （高 / 低 / 相同）。

表： salary

~~~sql
| id | employee_id | amount | pay_date   |
|----|-------------|--------|------------|
| 1  | 1           | 9000   | 2017-03-31 |
| 2  | 2           | 6000   | 2017-03-31 |
| 3  | 3           | 10000  | 2017-03-31 |
| 4  | 1           | 7000   | 2017-02-28 |
| 5  | 2           | 6000   | 2017-02-28 |
| 6  | 3           | 8000   | 2017-02-28 |
~~~

**employee_id** 字段是表 `employee` 中 **employee_id** 字段的外键。

~~~sql
| employee_id | department_id |
|-------------|---------------|
| 1           | 1             |
| 2           | 2             |
| 3           | 2             |
~~~

~~~ sql
| pay_month | department_id | comparison  |
|-----------|---------------|-------------|
| 2017-03   | 1             | higher      |
| 2017-03   | 2             | lower       |
| 2017-02   | 1             | same        |
| 2017-02   | 2             | same        |
~~~

在三月，公司的平均工资是 (9000+6000+10000)/3 = 8333.33...

由于部门 '1' 里只有一个 employee_id 为 '1' 的员工，所以部门 '1' 的平均工资就是此人的工资 9000 。因为 9000 > 8333.33 ，所以比较结果是 'higher'

第二个部门的平均工资为 employee_id 为 '2' 和 '3' 两个人的平均工资，为 (6000+10000)/2=8000 。因为 8000 < 8333.33 ，所以比较结果是 'lower' 。

方法：使用 `avg()` 和 `case...when...` [Accepted]

想法

通过如下 3 步解决这个问题。

算法

计算公司每个月的平均工资
MySQL 有一个内置函数 avg() 获得一列数字的平均值。同时我们需要将 pay_date 列按一定格式输出以便后面使用。

~~~sql
select avg(amount) as company_avg,  date_format(pay_date, '%Y-%m') as pay_month
from salary
group by date_format(pay_date, '%Y-%m')
;
~~~

| company_avg | pay_month |
| ----------- | --------- |
| 7000.0000   | 2017-02   |
| 8333.3333   | 2017-03   |

计算每个部门每个月的平均工资
为了实现这个目标，我们将 salary 表和 employee 表用条件 salary.employee_id = employee.id 连接起来。

~~~sql
select department_id, avg(amount) as department_avg, date_format(pay_date, '%Y-%m') as pay_month
from salary
join employee on salary.employee_id = employee.employee_id
group by department_id, pay_month
;
~~~

department_id	department_avg	pay_month
1	7000.0000	2017-02
1	9000.0000	2017-03
2	7000.0000	2017-02
2	8000.0000	2017-03

将 2 中的表和之前的公司数据作比较并求出结果
如果没用过 MySQL 的流控制语句 case...when... 那这一步会是最难的。
就像其他语言一样，MySQL 也有流控制语句，点击 这里 了解更多。

最后，将上面两个查询结合起来并用 on department_salary.pay_month = company_salary.pay_month 将它们连接。

~~~sql
select department_salary.pay_month, department_id,
case
  when department_avg>company_avg then 'higher'
  when department_avg<company_avg then 'lower'
  else 'same'
end as comparison
from
(
  select department_id, avg(amount) as department_avg, date_format(pay_date, '%Y-%m') as pay_month
  from salary join employee on salary.employee_id = employee.employee_id
  group by department_id, pay_month
) as department_salary
join
(
  select avg(amount) as company_avg,  date_format(pay_date, '%Y-%m') as pay_month from salary group by date_format(pay_date, '%Y-%m')
) as company_salary
on department_salary.pay_month = company_salary.pay_month;
~~~

> 开窗函数

需要注意两个开窗函数的条件，一个是只对日期分组，另一个对部门和日期同时分组

OVER(PARTITION BY pay_date) OVER(PARTITION BY employee.department_id, pay_date)

~~~sql
另外要注意结果的排序，首先按照日期降序，日期相同的按照部门ID升序排序
ORDER BY a.pay_date DESC, a.department_id ASC
然后就是条件的判断
CASE
    WHEN a.avg_salary1 > a.avg_salary2 THEN "higher"
    WHEN a.avg_salary1 < a.avg_salary2 THEN "lower"
    ELSE "same"
END AS "comparison"
~~~

~~~sql
CASE
    WHEN a.avg_salary1 > a.avg_salary2 THEN "higher"
    WHEN a.avg_salary1 < a.avg_salary2 THEN "lower"
    ELSE "same"
END AS "comparison"
~~~

最后要注意日期类型的格式，只保留年、月

~~~sql
SELECT DISTINCT date_format(a.pay_date, '%Y-%m') AS "pay_month",
       a.department_id AS "department_id",
       CASE
            WHEN a.avg_salary1 > a.avg_salary2 THEN "higher"
            WHEN a.avg_salary1 < a.avg_salary2 THEN "lower"
            ELSE "same"
       END AS "comparison"
FROM (
         SELECT 
                salary.pay_date,
                salary.amount,
                employee.department_id, AVG(amount) OVER(PARTITION BY pay_date) AS "avg_salary2",
                AVG(salary.amount) OVER(PARTITION BY employee.department_id, pay_date) AS "avg_salary1"
         FROM salary 
         INNER JOIN employee 
         ON salary.employee_id = employee.employee_id
     ) a
ORDER BY a.pay_date DESC, a.department_id ASC
~~~

