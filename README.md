# Leetcode-SQL-Database
Start from Dec 09 2021
Solving Database problems from leetcode 

# Date: Dec 09 2021
## 177. Nth Highest Salary
Write an SQL query to report the nth highest salary from the Employee table. If there is no nth highest salary, the query should report null.
My solution 
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
Declare m INT; 
set m=n-1;

  RETURN (
      # Write your MySQL query statement below.
      Select salary
      From Employee
      Group by salary 
      order by salary Desc
      Limit m,1
      
  );
END

Time/Efficiency:359 ms
Note:
-Limit(offset,count)
-Limit and order count from row0

# Date: Dec 15 2021
## 180. Consecutive numbers
Write an SQL query to find all numbers that appear at least three times consecutively.
My solution(wrong):
select num
from logs
where count(num)>3
Solution1: self-join
select distinct a.num as ConsecutiveNums
from logs a join logs b
on a.id=b.id+1 and a.num=b.num
Join logs c
on a.id=c.id+2 and a.num=c.num

Time/Efficiency:388 ms
Note:
-Format: 
 SELECT/FROM/ON need to use capital letters
 line 1: select/from 
 line 2: join on and 
-When it comes to compare with themselves/find consecutive numbers, need to use self-join.
-Everytime，when finishing writting code, need to look back and sort the logic to see if anything else need to be added in the query: eg. distinct

Solution2: window functions 
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT num,
        LEAD(num) OVER (ORDER BY id)  AS next,
        LAG(num) OVER (ORDER BY id)  AS prev
    FROM logs
    ) tb1
WHERE num = next AND next = prev

Time/Efficiency:533 ms
Note:
-Lead over partrition by and lag over partrition by 
 lead() to extract the next row of number
 lag() to extract the pre row of number

Solution 2.1:
with cte as 
(SELECT Num,
LEAD(Num,1) over (order by id) as first_lead,
LEAD(Num,2) over (order by id) as second_lead
from logs)

select distinct Num as ConsecutiveNums
from cte
where Num = first_lead and first_lead = second_lead

Solution3: 
Select distinct Num as ConsecutiveNums
from Logs
where (Id + 1, Num) in (select * from Logs) and (Id + 2, Num) in (select * from Logs)

# Date: Dec 16 2021
## 178. Rank Scores
Write an SQL query to rank the scores. The ranking should be calculated according to the following rules:
The scores should be ranked from the highest to the lowest.
If there is a tie between two scores, both should have the same ranking.
After a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no holes between ranks.
My solution(wrong):
SELECT Score, rank(Score) over (order by id ) AS ranking 
FROM Scores 
ORDER BY Score

Solution1:
SELECT score, dense_rank() over (order by score DESC) as 'rank'
FROM Scores

Note:
1. dense_rank/rank/row_number的用法区别： https://blog.csdn.net/u011726005/article/details/94592866
2. over（）括号里面应该排序，而不是在末尾加orderby
3. rank涉及keyword，因此要加引号

## 184. Department Highest Salary
Write an SQL query to find employees who have the highest salary in each of the departments.

My solution(wrong):
with cte as(SELECT rank() over(Partrition by Department order by salary DESC) as rank1
FROM Employee e
JOIN Department d
ON e.departmentId=d.id                              
) a

SELECT d.name AS Department, e.name AS Employee, e.salary as Salary
FROM Employee e
JOIN Department d
ON e.departmentId=d.id
WHERE a.rank1=1

My solution(correct)
select Department,Employee,Salary
from(
SELECT d.name AS Department, e.name AS Employee, e.salary as Salary, 
rank() over(partition by d.name order by e.salary DESC) as rn
FROM Employee e
JOIN Department d
ON e.departmentId=d.id
) a
where rn=1

Runtime: 502 ms
Notes:
-typo error: partition by 
-window function cannot be used in where clause

## 185. Department Top Three Salaries
My solution(correct)
select Department,Employee,Salary
from(
SELECT d.name AS Department, e.name AS Employee, e.salary as Salary, 
dense_rank() over(partition by d.name order by e.salary DESC) as rn
FROM Employee e
JOIN Department d
ON e.departmentId=d.id
) a
where rn<=3
Runtime: 149 ms

?question:为什么用rank不能accept，结果是一样的

## 1511. Customer Order Frequency
Write an SQL query to report the customer_id and customer_name of customers who have spent at least $100 in each month of June and July 2020.
My answer(wrong):
SELECT c.customer_id,c.name, sum(quantity*price) as total
FROM customers c
JOIN orders o ON c.customer_id=o.customer_id 
JOIN product p ON p.product_id=o.product_id
Where substring(order_date,5,2) in ('06','07') 
Group by customer_id,name
having total>=100

Notes:
1.having/where/group by的顺序
2.如何提取日期的月份，天，以及年

solution1:
SELECT customer_id, name
FROM (
    SELECT c.customer_id, c.name, 
    SUM(CASE WHEN LEFT(o.order_date, 7) = '2020-06' THEN p.price*o.quantity ELSE 0 END) AS t1, 
    SUM(CASE WHEN LEFT(o.order_date, 7) = '2020-07' THEN p.price*o.quantity ELSE 0 END) AS t2
    FROM customers c
    JOIN orders o
    ON c.customer_id = o.customer_id
    JOIN product p
    ON p.product_id = o.product_id
    GROUP BY 1
    ) tmp
WHERE t1 >= 100 AND t2 >= 100

？question：为什么这里只group by 1个量，不是应该除了sum之外都需要被group by吗？
notes：
1.使用left来截取日期函数

# Date: Dec 17
## 577. Employee Bonus
Write an SQL query to report the name and bonus amount of each employee with a bonus less than 1000.
My solution(wrong):
SELECT e.name, b.bonus 
FROM employee e
JOIN bonus b
ON e.empId=b.empId
where b.bonus<1000 

Solution1:
SELECT e.name, b.bonus 
FROM employee e
LEFT JOIN bonus b
ON e.empId=b.empId
where b.bonus<1000 or b.bonus IS NULL

Note:
-Why use left join here: because we want to retain the employee info that even though they do not have bonus
-forgot to add 'b.bonus IS NULL' 没有考虑到如果bonus没有的情况

## 584. Find Customer Referee
My Answer(accepted):
SELECT C.name
FROM Customer C
WHERE c.referee_id !=2 or c.referee_id IS NULL

Solution1:
SELECT name 
FROM customer
WHERE referee_id <> 2 OR referee_id IS NULL
 
Note:
- 想要表示不等于，可以使用<> 或者！=

## 586. Customer Placing the Largest Number of Orders
My Answer:
SELECT customer_number
FROM 
(
SELECT customer_number, count(customer_number)
FROM Orders
group by customer_number
order by 2 DESC
LIMIT 1)a

Notes:
- 最初版本没有加上group by，说明对count函数理解不到位

## 597. Friend Requests I: Overall Acceptance Rate
My answer:
SELECT count(accepter_id)/count(sender_id)
FROM FriendRequest f
JOIN RequestAccepted r ON sender_id=requester_id

Notes:
-没理解题意
-follow up question2较难，没做

Solution1:
https://leetcode.com/problems/friend-requests-i-overall-acceptance-rate/discuss/358575/Detailed-Explaination-for-Question-and-2-follow-up
http://lixinchengdu.github.io/algorithmbook/leetcode/friend-requests-i-overall-acceptance-rate.html

## 603. Consecutive Available Seats
My Answer(wrong):
using lead() and lag() function
SELET seat_id
FROM 
(SELECT seat_id,
 lead(seat_id) over (order by seat_id) as nex,
 lag(seat_id) over (order by seat_id) as pre
 from cinema
 where next=seat_id+1,pre=seat_id-1
)a
WHERE free = 1
Order by seat_id ASC

Notes:
-没发现哪里出了问题

Solution1:
SELECT seat_id
FROM cinema
WHERE
    free = 1
    AND
    (
        seat_id + 1 IN (
            SELECT seat_id FROM cinema WHERE free = 1
        )
        OR
        seat_id - 1 IN (
            SELECT seat_id FROM cinema WHERE free = 1
        )
    )
;

Notes:
-在where clause里面加上subquery并不需要加上alias

## 607. Sales Person
My Answer(wrong):
SELECT s.name 
FROM salesperson s,Company c,orders o 
On s.sales_id=o.sales_id,c.com_id=o.com_id
where c.name<>'RED'

Notes:
-没有找到出错的原因













