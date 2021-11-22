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
select dt as login_date, count(*) as user_count
from(
    select user_id,min(activity_date) as dt
    from traffic
    where activity = 'login'
    group by user_id
    having min(activity_date) >= date_sub('2019-06-30',interval 90 day) ) as t
group by dt
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



#### [1303. 求团队人数](https://leetcode-cn.com/problems/find-the-team-size/)

~~~mysql
Employee Table:
+-------------+------------+
| employee_id | team_id    |
+-------------+------------+
|     1       |     8      |
|     2       |     8      |
|     3       |     8      |
|     4       |     7      |
|     5       |     9      |
|     6       |     9      |
+-------------+------------+
Result table:
+-------------+------------+
| employee_id | team_size  |
+-------------+------------+
|     1       |     3      |
|     2       |     3      |
|     3       |     3      |
|     4       |     1      |
|     5       |     2      |
|     6       |     2      |
+-------------+------------+
ID 为 1、2、3 的员工是 team_id 为 8 的团队的成员，
ID 为 4 的员工是 team_id 为 7 的团队的成员，
ID 为 5、6 的员工是 team_id 为 9 的团队的成员。
~~~

`自连接`

~~~mysql
# Self Join
SELECT e1.employee_id, COUNT(*) AS team_size
FROM Employee e1 JOIN Employee e2 USING (team_id)
GROUP BY e1.employee_id
ORDER BY e1.employee_id;
~~~

`相关子查询`

~~~mysql
# Correlated Subquery 
SELECT employee_id, (
    SELECT COUNT(*)
    FROM employee e2
    WHERE e1.team_id = e2.team_id
) AS team_size
FROM Employee e1
ORDER BY e1.employee_id;
~~~

`窗口函数`

```sql
# Window Function
SELECT
    employee_id,
    COUNT(employee_id) OVER(PARTITION BY team_id) AS team_size
FROM Employee
ORDER BY employee_id;
```



#### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

`Employee` 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

~~~mysql
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

~~~sql
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
~~~

~~~sql
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
~~~

`窗口函数的运用`

~~~sql
select Department, Employee, Salary
from 
(
select 
    D.Name as Department, 
    E.Name as Employee, 
    E.Salary as Salary, 
    rank() over(partition by D.Name order by E.Salary desc) as rank_
from Employee E join Department D on E.DepartmentId = D.Id
) as tmp
where rank_ = 1
~~~

方法：使用 `JOIN` 和 `IN` 语句

因为 **Employee** 表包含 *Salary* 和 *DepartmentId* 字段，我们可以以此在部门内查询最高工资。

SELECT    DepartmentId, MAX(Salary) FROM    Employee GROUP BY DepartmentId;

~~~sql
SELECT
    DepartmentId, MAX(Salary)
FROM
    Employee
GROUP BY DepartmentId;
~~~

> 注意：有可能有多个员工同时拥有最高工资，所以最好在这个查询中不包含雇员名字的信息。

~~~sql
| DepartmentId | MAX(Salary) |
|--------------|-------------|
| 1            | 90000       |
| 2            | 80000       |
~~~

然后，我们可以把表 **Employee** 和 **Department** 连接，再在这张临时表里用 `IN` 语句查询部门名字和工资的关系。

~~~sql
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
	)
;
~~~

~~~sql
| Department | Employee | Salary |
|------------|----------|--------|
| Sales      | Henry    | 80000  |
| IT         | Max      | 90000  |
~~~

