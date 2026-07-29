# LeetCode Top SQL 50 — Study Plan Reference

Source: https://leetcode.com/studyplan/top-sql-50/

Structure of this doc:
1. **Part 1** — All 50 questions listed by category
2. **Part 2** — Pattern / logic for each question (how to *think* about it)
3. **Part 3** — Full SQL solutions for each question

---

## PART 1 — The 50 Questions (by category)

### A. Select (4)
1. 1757. Recyclable and Low Fat Products
2. 584. Find Customer Referee
3. 595. Big Countries
4. 1148. Article Views I

### B. Basic Joins (10)
5. 1683. Invalid Tweets
6. 1378. Replace Employee ID With The Unique Identifier
7. 1068. Product Sales Analysis I
8. 1581. Customer Who Visited but Did Not Make Any Transactions
9. 197. Rising Temperature
10. 1661. Average Time of Process per Machine
11. 577. Employee Bonus
12. 1280. Students and Examinations
13. 570. Managers with at Least 5 Direct Reports
14. 1934. Confirmation Rate

### C. Basic Aggregate Functions (8)
15. 620. Not Boring Movies
16. 1251. Average Selling Price
17. 1075. Project Employees I
18. 1633. Percentage of Users Attended a Contest
19. 1211. Queries Quality and Percentage
20. 1193. Monthly Transactions I
21. 1174. Immediate Food Delivery II
22. 550. Game Play Analysis IV

### D. Sorting and Grouping (6)
23. 2356. Number of Unique Subjects Taught by Each Teacher
24. 1141. User Activity for the Past 30 Days I
25. 1070. Product Sales Analysis III
26. 596. Classes More Than 5 Students
27. 1729. Find Followers Count
28. 619. Biggest Single Number

### E. Advanced Select and Joins (4)
29. 1045. Customers Who Bought All Products
30. 1731. The Number of Employees Which Report to Each Employee
31. 1789. Primary Department for Each Employee
32. 610. Triangle Judgement

### F. Subqueries (4)
33. 180. Consecutive Numbers
34. 1164. Product Price at a Given Date
35. 1204. Last Person to Fit in the Bus
36. 1907. Count Salary Categories

### G. Advanced String Functions / Regex / Clause (14)
37. 1978. Employees Whose Manager Left the Company
38. 626. Exchange Seats
39. 1341. Movie Rating
40. 1321. Restaurant Growth
41. 602. Friend Requests II: Who Has the Most Friends
42. 585. Investments in 2016
43. 185. Department Top Three Salaries
44. 1667. Fix Names in a Table
45. 1527. Patients With a Condition
46. 196. Delete Duplicate Emails
47. 176. Second Highest Salary
48. 1517. Find Users With Valid E-Mails
49. 2199. Group Sold Products By The Date
50. 2205. List the Products Ordered in a Period

---

## PART 2 — Pattern / Logic for Each Question

### A. Select

**1. Recyclable and Low Fat Products** — Straight `WHERE` filter with two equality conditions AND'd together. No joins, no aggregation.

**2. Find Customer Referee** — Filter out rows matching a value, *including* NULLs. The trap: `!= 2` silently drops NULL rows, so you need `IS NULL OR != 2`.

**3. Big Countries** — Filter with `OR` across two independent conditions (area OR population threshold), not AND.

**4. Article Views I** — Self-referential filter: author viewing their own article means `author_id = viewer_id`. Need `DISTINCT` and `ORDER BY`.

### B. Basic Joins

**5. Invalid Tweets** — No join needed at all — it's a `WHERE LENGTH(...) > 15` filter. Included in "joins" section mostly as a warm-up.

**6. Replace Employee ID With The Unique Identifier** — `LEFT JOIN` (not INNER) because employees without a unique ID must still appear, with NULL.

**7. Product Sales Analysis I** — Simple `INNER JOIN` on product_id to pull a column (product_name) from a lookup table.

**8. Customer Who Visited but Did Not Make Any Transactions** — Anti-join pattern: `LEFT JOIN ... WHERE right.key IS NULL`, then `GROUP BY` to count.

**9. Rising Temperature** — Self-join on `date = date - INTERVAL 1 DAY` (date-arithmetic self join), then compare temperatures.

**10. Average Time of Process per Machine** — Self-join same table on machine_id + process_id, pairing `activity_type='start'` row with `activity_type='end'` row, subtract timestamps, average.

**11. Employee Bonus** — `LEFT JOIN` + `WHERE bonus < 1000 OR bonus IS NULL` (again, the NULL trap).

**12. Students and Examinations** — Cartesian `CROSS JOIN` of Students × Subjects to get every possible pairing, `LEFT JOIN` to actual Examinations, `COUNT` non-null attendance rows, `GROUP BY` all four id/name columns.

**13. Managers with at Least 5 Direct Reports** — Self-join Employee to itself (report.managerId = manager.id), `GROUP BY` manager, `HAVING COUNT(*) >= 5`.

**14. Confirmation Rate** — `LEFT JOIN` Signups to Confirmations, aggregate with `AVG(CASE WHEN action='confirmed' THEN 1 ELSE 0 END)`, `ROUND` to 2 decimals, and default to 0 for users with no confirmation rows (LEFT JOIN + AVG over 0 rows or all-null handled via ROUND(IFNULL(...,0),2)).

### C. Basic Aggregate Functions

**15. Not Boring Movies** — Filter (`id % 2 = 1`, description not 'boring') then `ORDER BY rating DESC`. Pure filtering, no aggregation despite the section name.

**16. Average Selling Price** — `LEFT JOIN` Prices to UnitsSold on product_id AND date range (`purchase_date BETWEEN start_date AND end_date`), `SUM(units*price)/SUM(units)`, `GROUP BY` product, handle products with 0 sales via `ROUND(IFNULL(...,0),2)`.

**17. Project Employees I** — `JOIN` + `GROUP BY project_id` + `AVG(experience_years)`, rounded.

**18. Percentage of Users Attended a Contest** — `COUNT(DISTINCT user_id) / (SELECT COUNT(*) FROM Users) * 100`, `GROUP BY contest_id`, `ORDER BY` percentage desc then contest_id.

**19. Queries Quality and Percentage** — Two aggregates in one `GROUP BY query_name`: `AVG(rating/position)` for quality, and `AVG(CASE WHEN rating<3 THEN 1 ELSE 0 END)*100` for poor-query percentage.

**20. Monthly Transactions I** — `GROUP BY` a computed month (`DATE_FORMAT(trans_date,'%Y-%m')`) and `country`, aggregate `COUNT`, `SUM`, and conditional `SUM(CASE WHEN state='approved'...)`.

**21. Immediate Food Delivery II** — Identify each customer's *first* order (`MIN(order_date)` per customer via subquery/window), then compute the % where order_date = customer_pref_delivery_date.

**22. Game Play Analysis IV** — Find each player's first login date, then check (via a correlated `EXISTS`/join) whether they also logged in on first_login + 1 day; divide count by total distinct players.

### D. Sorting and Grouping

**23. Number of Unique Subjects Taught by Each Teacher** — `GROUP BY teacher_id`, `COUNT(DISTINCT subject_id)`.

**24. User Activity for the Past 30 Days I** — Date-range filter (`activity_date BETWEEN end_date-29 AND end_date`), `COUNT(DISTINCT user_id)` `GROUP BY` day.

**25. Product Sales Analysis III** — Find each product's *first* sale year (subquery with `MIN(year)` grouped by product), then join back to get quantity/price for exactly that year.

**26. Classes More Than 5 Students** — `GROUP BY class`, `HAVING COUNT(DISTINCT student) >= 5`.

**27. Find Followers Count** — `GROUP BY user_id`, `COUNT(follower_id)`, `ORDER BY user_id`.

**28. Biggest Single Number** — Find numbers that appear exactly once (`GROUP BY num HAVING COUNT(*)=1`), then take `MAX` of those (wrapped so it returns NULL, not empty, if none exist).

### E. Advanced Select and Joins

**29. Customers Who Bought All Products** — Classic "division" pattern: `GROUP BY customer_id`, `HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product)`.

**30. The Number of Employees Which Report to Each Employee** — Self-join, `GROUP BY` manager id/name, `COUNT(*)` reports, `AVG(age)` rounded, filter to only managers who actually have reports (inner join naturally does this).

**31. Primary Department for Each Employee** — `UNION` of two sets: employees who belong to only one department (no explicit 'Y' flag needed, found via `HAVING COUNT(*)=1`), and employees whose primary flag is 'Y'.

**32. Triangle Judgement** — Row-wise `CASE WHEN` applying the triangle inequality (sum of any two sides > third) — no joins/aggregation, just conditional logic per row.

### F. Subqueries

**33. Consecutive Numbers** — Compare each row to the next two using self-joins on `id+1`/`id+2` (or `LAG`/`LEAD` window functions), keep rows where all three nums are equal.

**34. Product Price at a Given Date** — For each product find the latest change_date ≤ target date (subquery with `MAX(change_date)` filtered), `UNION` with products that had no price change before that date (default price 10).

**35. Last Person to Fit in the Bus** — Running total pattern: compute cumulative `SUM(weight)` ordered by turn (window function `SUM() OVER`), find max turn where cumulative ≤ 1000.

**36. Count Salary Categories** — Bucket amounts via `CASE WHEN` into 3 named ranges, `COUNT` per bucket, then `UNION ALL` so all three categories always appear even with 0 count (subquery counts can't just GROUP BY because empty buckets would vanish).

### G. Advanced String Functions / Regex / Clause

**37. Employees Whose Manager Left the Company** — Filter employees whose `manager_id` is not null AND not found in the set of existing employee ids (`NOT IN` subquery), plus salary condition.

**38. Exchange Seats** — Swap adjacent odd/even ids using `CASE WHEN` with `id+1`/`id-1`, keep last odd id alone if odd count of rows (`LEAD` window function is the modern clean way).

**39. Movie Rating** — Two separate ranked aggregates unioned: user who rated the most movies (`GROUP BY user_id`, top by count then name), and movie with highest average rating in Feb 2020 (`GROUP BY movie_id`, top by avg then title).

**40. Restaurant Growth** — 7-day rolling window: for each date, `SUM` of amount and `COUNT(DISTINCT customer)` over the preceding 6 days + current day (self-join on date range or window function), only from the 7th day onward.

**41. Friend Requests II: Who Has the Most Friends** — Treat each accepted request as two directed "friend" edges (requester→accepter and accepter→requester) via `UNION ALL`, then `GROUP BY id`, `COUNT`, `ORDER BY` desc, `LIMIT 1`.

**42. Investments in 2016** — Filter people whose `tiv_2015` value is shared with at least one other person (duplicate check via window `COUNT() OVER` or subquery), AND whose (lat,lon) pair is unique (no one else at exact same location), then `SUM(tiv_2016)`.

**43. Department Top Three Salaries** — Dense ranking within each department (`DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC)`), keep rank ≤ 3.

**44. Fix Names in a Table** — String manipulation: `UPPER(LEFT(name,1))` concatenated with `LOWER(SUBSTRING(name,2))`, `ORDER BY user_id`.

**45. Patients With a Condition** — Pattern match on a space-delimited code using `LIKE 'DIAB1%' OR LIKE '% DIAB1%'` (must match code as a whole token, not a substring inside another code).

**46. Delete Duplicate Emails** — `DELETE` (not select) — keep the lowest id per duplicate email using a self-join or correlated subquery comparing ids.

**47. Second Highest Salary** — Distinct salaries, order desc, offset 1 (`LIMIT 1 OFFSET 1` or `DENSE_RANK`), wrapped so it returns NULL instead of empty when there's no second value.

**48. Find Users With Valid E-Mails** — Regex match (`REGEXP`) enforcing: starts with a letter, only allowed characters before `@`, domain exactly `@leetcode.com`, nothing after `.com`.

**49. Group Sold Products By The Date** — `GROUP BY sell_date`, `COUNT(DISTINCT product)` as num_sold, and a comma-joined sorted list of distinct product names (`GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',')`).

**50. List the Products Ordered in a Period** — Join Products to Orders, filter order_date to the given month/year, `GROUP BY` product, `HAVING SUM(unit)>=100`.

---

## PART 3 — Solutions

### A. Select

**1. Recyclable and Low Fat Products**
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y';
```

**2. Find Customer Referee**
```sql
SELECT name
FROM Customer
WHERE referee_id IS NULL OR referee_id != 2;
```

**3. Big Countries**
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

**4. Article Views I**
```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY id;
```

### B. Basic Joins

**5. Invalid Tweets**
```sql
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15;
```

**6. Replace Employee ID With The Unique Identifier**
```sql
SELECT eu.unique_id, e.name
FROM Employees e
LEFT JOIN EmployeeUNI eu ON e.id = eu.id;
```

**7. Product Sales Analysis I**
```sql
SELECT p.product_name, s.year, s.price
FROM Sales s
JOIN Product p ON s.product_id = p.product_id;
```

**8. Customer Who Visited but Did Not Make Any Transactions**
```sql
SELECT v.customer_id, COUNT(*) AS count_no_trans
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE t.transaction_id IS NULL
GROUP BY v.customer_id;
```

**9. Rising Temperature**
```sql
SELECT w2.id
FROM Weather w1
JOIN Weather w2 ON w2.recordDate = DATE_ADD(w1.recordDate, INTERVAL 1 DAY)
WHERE w2.temperature > w1.temperature;
```

**10. Average Time of Process per Machine**
```sql
SELECT s.machine_id,
       ROUND(AVG(e.timestamp - s.timestamp), 3) AS processing_time
FROM Activity s
JOIN Activity e
  ON s.machine_id = e.machine_id
 AND s.process_id = e.process_id
 AND s.activity_type = 'start'
 AND e.activity_type = 'end'
GROUP BY s.machine_id;
```

**11. Employee Bonus**
```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

**12. Students and Examinations**
```sql
SELECT s.student_id, s.student_name, sub.subject_name,
       COUNT(e.subject_name) AS attended_exams
FROM Students s
CROSS JOIN Subjects sub
LEFT JOIN Examinations e
  ON s.student_id = e.student_id AND sub.subject_name = e.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```

**13. Managers with at Least 5 Direct Reports**
```sql
SELECT m.name
FROM Employee e
JOIN Employee m ON e.managerId = m.id
GROUP BY m.id, m.name
HAVING COUNT(*) >= 5;
```

**14. Confirmation Rate**
```sql
SELECT s.user_id,
       ROUND(IFNULL(AVG(c.action = 'confirmed'), 0), 2) AS confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c ON s.user_id = c.user_id
GROUP BY s.user_id;
```

### C. Basic Aggregate Functions

**15. Not Boring Movies**
```sql
SELECT *
FROM Cinema
WHERE id % 2 = 1 AND description != 'boring'
ORDER BY rating DESC;
```

**16. Average Selling Price**
```sql
SELECT p.product_id,
       ROUND(IFNULL(SUM(u.units * p.price) / SUM(u.units), 0), 2) AS average_price
FROM Prices p
LEFT JOIN UnitsSold u
  ON p.product_id = u.product_id
 AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

**17. Project Employees I**
```sql
SELECT p.project_id, ROUND(AVG(e.experience_years), 2) AS average_years
FROM Project p
JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

**18. Percentage of Users Attended a Contest**
```sql
SELECT contest_id,
       ROUND(COUNT(DISTINCT user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) AS percentage
FROM Register
GROUP BY contest_id
ORDER BY percentage DESC, contest_id;
```

**19. Queries Quality and Percentage**
```sql
SELECT query_name,
       ROUND(AVG(rating / position), 2) AS quality,
       ROUND(AVG(rating < 3) * 100, 2) AS poor_query_percentage
FROM Queries
GROUP BY query_name;
```

**20. Monthly Transactions I**
```sql
SELECT DATE_FORMAT(trans_date, '%Y-%m') AS month, country,
       COUNT(*) AS trans_count,
       SUM(state = 'approved') AS approved_count,
       SUM(amount) AS trans_total_amount,
       SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount
FROM Transactions
GROUP BY month, country;
```

**21. Immediate Food Delivery II**
```sql
SELECT ROUND(
  100.0 * SUM(order_date = customer_pref_delivery_date) / COUNT(*), 2
) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date) IN (
  SELECT customer_id, MIN(order_date)
  FROM Delivery
  GROUP BY customer_id
);
```

**22. Game Play Analysis IV**
```sql
WITH first_login AS (
  SELECT player_id, MIN(event_date) AS first_date
  FROM Activity
  GROUP BY player_id
)
SELECT ROUND(
  COUNT(DISTINCT a.player_id) / (SELECT COUNT(*) FROM first_login), 2
) AS fraction
FROM first_login f
JOIN Activity a
  ON a.player_id = f.player_id
 AND a.event_date = DATE_ADD(f.first_date, INTERVAL 1 DAY);
```

### D. Sorting and Grouping

**23. Number of Unique Subjects Taught by Each Teacher**
```sql
SELECT teacher_id, COUNT(DISTINCT subject_id) AS cnt
FROM Teacher
GROUP BY teacher_id;
```

**24. User Activity for the Past 30 Days I**
```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27'
GROUP BY activity_date;
```

**25. Product Sales Analysis III**
```sql
SELECT s.product_id, s.year AS first_year, s.quantity, s.price
FROM Sales s
JOIN (
  SELECT product_id, MIN(year) AS first_year
  FROM Sales
  GROUP BY product_id
) f ON s.product_id = f.product_id AND s.year = f.first_year;
```

**26. Classes More Than 5 Students**
```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5;
```

**27. Find Followers Count**
```sql
SELECT user_id, COUNT(follower_id) AS followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

**28. Biggest Single Number**
```sql
SELECT MAX(num) AS num
FROM (
  SELECT num
  FROM MyNumbers
  GROUP BY num
  HAVING COUNT(*) = 1
) t;
```

### E. Advanced Select and Joins

**29. Customers Who Bought All Products**
```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
```

**30. The Number of Employees Which Report to Each Employee**
```sql
SELECT m.employee_id, m.name,
       COUNT(e.employee_id) AS reports_count,
       ROUND(AVG(e.age), 0) AS average_age
FROM Employees e
JOIN Employees m ON e.reports_to = m.employee_id
GROUP BY m.employee_id, m.name
ORDER BY m.employee_id;
```

**31. Primary Department for Each Employee**
```sql
SELECT employee_id, department_id
FROM Employee
GROUP BY employee_id
HAVING COUNT(*) = 1
UNION
SELECT employee_id, department_id
FROM Employee
WHERE primary_flag = 'Y';
```

**32. Triangle Judgement**
```sql
SELECT x, y, z,
  CASE WHEN x + y > z AND x + z > y AND y + z > x
       THEN 'Yes' ELSE 'No' END AS triangle
FROM Triangle;
```

### F. Subqueries

**33. Consecutive Numbers**
```sql
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l1.id + 1 = l2.id
JOIN Logs l3 ON l1.id + 2 = l3.id
WHERE l1.num = l2.num AND l2.num = l3.num;
```

**34. Product Price at a Given Date**
```sql
SELECT product_id, 10 AS price
FROM Products
GROUP BY product_id
HAVING MIN(change_date) > '2019-08-16'
UNION
SELECT product_id, new_price AS price
FROM Products
WHERE (product_id, change_date) IN (
  SELECT product_id, MAX(change_date)
  FROM Products
  WHERE change_date <= '2019-08-16'
  GROUP BY product_id
);
```

**35. Last Person to Fit in the Bus**
```sql
SELECT person_name
FROM (
  SELECT person_name,
         SUM(weight) OVER (ORDER BY turn) AS running_weight
  FROM Queue
) t
WHERE running_weight <= 1000
ORDER BY running_weight DESC
LIMIT 1;
```

**36. Count Salary Categories**
```sql
SELECT 'Low Salary' AS category,
       SUM(income < 20000) AS accounts_count
FROM Accounts
UNION ALL
SELECT 'Average Salary',
       SUM(income BETWEEN 20000 AND 50000)
FROM Accounts
UNION ALL
SELECT 'High Salary',
       SUM(income > 50000)
FROM Accounts;
```

### G. Advanced String Functions / Regex / Clause

**37. Employees Whose Manager Left the Company**
```sql
SELECT employee_id
FROM Employees
WHERE salary < 30000
  AND manager_id IS NOT NULL
  AND manager_id NOT IN (SELECT employee_id FROM Employees)
ORDER BY employee_id;
```

**38. Exchange Seats**
```sql
SELECT
  CASE
    WHEN id % 2 = 1 AND id = (SELECT MAX(id) FROM Seat) THEN id
    WHEN id % 2 = 1 THEN id + 1
    ELSE id - 1
  END AS id,
  student
FROM Seat
ORDER BY id;
```

**39. Movie Rating**
```sql
(SELECT u.name AS results
 FROM MovieRating mr
 JOIN Users u ON mr.user_id = u.user_id
 GROUP BY mr.user_id, u.name
 ORDER BY COUNT(*) DESC, u.name ASC
 LIMIT 1)
UNION ALL
(SELECT m.title AS results
 FROM MovieRating mr
 JOIN Movies m ON mr.movie_id = m.movie_id
 WHERE mr.created_at BETWEEN '2020-02-01' AND '2020-02-29'
 GROUP BY mr.movie_id, m.title
 ORDER BY AVG(mr.rating) DESC, m.title ASC
 LIMIT 1);
```

**40. Restaurant Growth**
```sql
SELECT visited_on,
       SUM(amount) AS amount,
       ROUND(SUM(amount) / 7, 2) AS average_amount
FROM (
  SELECT c1.visited_on,
         (SELECT SUM(amount) FROM Customer c2
          WHERE c2.visited_on BETWEEN DATE_SUB(c1.visited_on, INTERVAL 6 DAY) AND c1.visited_on) AS amount
  FROM Customer c1
  GROUP BY c1.visited_on
  HAVING visited_on >= (SELECT DATE_ADD(MIN(visited_on), INTERVAL 6 DAY) FROM Customer)
) t
GROUP BY visited_on, amount
ORDER BY visited_on;
```

**41. Friend Requests II: Who Has the Most Friends**
```sql
SELECT id, COUNT(*) AS num
FROM (
  SELECT requester_id AS id FROM RequestAccepted
  UNION ALL
  SELECT accepter_id AS id FROM RequestAccepted
) t
GROUP BY id
ORDER BY num DESC
LIMIT 1;
```

**42. Investments in 2016**
```sql
SELECT ROUND(SUM(tiv_2016), 2) AS tiv_2016
FROM Insurance
WHERE tiv_2015 IN (
  SELECT tiv_2015 FROM Insurance GROUP BY tiv_2015 HAVING COUNT(*) > 1
)
AND (lat, lon) IN (
  SELECT lat, lon FROM Insurance GROUP BY lat, lon HAVING COUNT(*) = 1
);
```

**43. Department Top Three Salaries**
```sql
SELECT d.name AS Department, t.name AS Employee, t.salary AS Salary
FROM (
  SELECT name, salary, departmentId,
         DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rk
  FROM Employee
) t
JOIN Department d ON t.departmentId = d.id
WHERE t.rk <= 3;
```

**44. Fix Names in a Table**
```sql
SELECT user_id,
       CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2))) AS name
FROM Users
ORDER BY user_id;
```

**45. Patients With a Condition**
```sql
SELECT patient_id, patient_name, conditions
FROM Patients
WHERE conditions LIKE 'DIAB1%' OR conditions LIKE '% DIAB1%';
```

**46. Delete Duplicate Emails**
```sql
DELETE p1
FROM Person p1
JOIN Person p2
  ON p1.email = p2.email AND p1.id > p2.id;
```

**47. Second Highest Salary**
```sql
SELECT (
  SELECT DISTINCT salary
  FROM Employee
  ORDER BY salary DESC
  LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

**48. Find Users With Valid E-Mails**
```sql
SELECT *
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode\\.com$';
```

**49. Group Sold Products By The Date**
```sql
SELECT sell_date,
       COUNT(DISTINCT product) AS num_sold,
       GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

**50. List the Products Ordered in a Period**
```sql
SELECT p.product_name, SUM(o.unit) AS unit
FROM Products p
JOIN Orders o ON p.product_id = o.product_id
WHERE o.order_date BETWEEN '2020-02-01' AND '2020-02-29'
GROUP BY p.product_name
HAVING SUM(o.unit) >= 100;
```

---

*Note: MySQL is the dialect used above (LeetCode's default). A few problems (window-function ones especially) also have valid Postgres/Oracle/MS SQL syntax — flag it if you want those variants too.*
