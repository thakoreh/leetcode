# [2854. Rolling Average Steps](https://leetcode.cn/problems/rolling-average-steps)

[English Version](/solution/2800-2899/2854.Rolling%20Average%20Steps/README_EN.md)

## 题目描述

<!-- 这里写题目描述 -->

<p>Table: <code><font face="monospace">Steps</font></code></p>

<pre>
+-------------+------+ 
| Column Name | Type | 
+-------------+------+ 
| user_id     | int  | 
| steps_count | int  |
| steps_date  | date |
+-------------+------+
(user_id, steps_date) is the primary key for this table.
Each row of this table contains user_id, steps_count, and steps_date.
</pre>

<p>Write a solution to calculate <code>3-day</code> <strong>rolling averages</strong> of steps for each user.</p>

<p>We calculate the <code>n-day</code> <strong>rolling average</strong> this way:</p>

<ul>
	<li>For each day, we calculate the average of <code>n</code> consecutive days of step counts ending on that day if available, otherwise, <code>n-day</code> rolling average is not defined for it.</li>
</ul>

<p>Output the <code>user_id</code>, <code>steps_date</code>, and rolling average. Round the rolling average to <strong>two decimal places</strong>.</p>

<p>Return<em> the result table ordered by </em><code>user_id</code><em>, </em><code>steps_date</code><em> in <strong>ascending</strong> order.</em></p>

<p>The result format is in the following example.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre>
<strong>Input:</strong> 
Steps table:
+---------+-------------+------------+
| user_id | steps_count | steps_date |
+---------+-------------+------------+
| 1       | 687         | 2021-09-02 |
| 1       | 395         | 2021-09-04 |
| 1       | 499         | 2021-09-05 |
| 1       | 712         | 2021-09-06 |
| 1       | 576         | 2021-09-07 |
| 2       | 153         | 2021-09-06 |
| 2       | 171         | 2021-09-07 |
| 2       | 530         | 2021-09-08 |
| 3       | 945         | 2021-09-04 |
| 3       | 120         | 2021-09-07 |
| 3       | 557         | 2021-09-08 |
| 3       | 840         | 2021-09-09 |
| 3       | 627         | 2021-09-10 |
| 5       | 382         | 2021-09-05 |
| 6       | 480         | 2021-09-01 |
| 6       | 191         | 2021-09-02 |
| 6       | 303         | 2021-09-05 |
+---------+-------------+------------+
<strong>Output:</strong> 
+---------+------------+-----------------+
| user_id | steps_date | rolling_average | 
+---------+------------+-----------------+
| 1       | 2021-09-06 | 535.33          | 
| 1       | 2021-09-07 | 595.67          | 
| 2       | 2021-09-08 | 284.67          |
| 3       | 2021-09-09 | 505.67          |
| 3       | 2021-09-10 | 674.67          |    
+---------+------------+-----------------+
<strong>Explanation:</strong> 
- For user id 1, the step counts for the three consecutive days up to 2021-09-06 are available. Consequently, the rolling average for this particular date is computed as (395 + 499 + 712) / 3 = 535.33.
- For user id 1, the step counts for the three consecutive days up to 2021-09-07 are available. Consequently, the rolling average for this particular date is computed as (499 + 712 + 576) / 3 = 595.67.
- For user id 2, the step counts for the three consecutive days up to 2021-09-08 are available. Consequently, the rolling average for this particular date is computed as (153 + 171 + 530) / 3 = 284.67.
- For user id 3, the step counts for the three consecutive days up to 2021-09-09 are available. Consequently, the rolling average for this particular date is computed as (120 + 557 + 840) / 3 = 505.67.
- For user id 3, the step counts for the three consecutive days up to 2021-09-10 are available. Consequently, the rolling average for this particular date is computed as (557 + 840 + 627) / 3 = 674.67.
- For user id 4 and 5, the calculation of the rolling average is not viable as there is insufficient data for the consecutive three days. Output table ordered by user_id and steps_date in ascending order.</pre>

## 解法

<!-- 这里可写通用的实现逻辑 -->

**方法一：窗口函数**

我们用窗口函数 `LAG() OVER()` 来计算每个用户当前日期与上上个日期之间的天数差，如果为 $2$，说明这两个日期之间有连续 $3$ 天的数据，我们可以利用窗口函数 `AVG() OVER()` 来计算这 $3$ 个数据的平均值。

<!-- tabs:start -->

### **SQL**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```sql
# Write your MySQL query statement below
WITH
    T AS (
        SELECT
            user_id,
            steps_date,
            round(
                avg(steps_count) OVER (
                    PARTITION BY user_id
                    ORDER BY steps_date
                    ROWS 2 PRECEDING
                ),
                2
            ) AS rolling_average,
            datediff(
                steps_date,
                lag(steps_date, 2) OVER (
                    PARTITION BY user_id
                    ORDER BY steps_date
                )
            ) = 2 AS st
        FROM Steps
    )
SELECT
    user_id,
    steps_date,
    rolling_average
FROM T
WHERE st = 1
ORDER BY 1, 2;
```

<!-- tabs:end -->
