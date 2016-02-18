# Analytic Functions
[Oracle Analytic Functions Reference Page](http://docs.oracle.com/cd/E11882_01/server.112/e41084/functions004.htm#SQLRF06174)
```
Analytic Functions or Window Functions compute an aggregate value based on a group of rows.
```
The group of rows is called a **window** and that is how they are different from just an aggregate function. A simple aggregate function doesn't let you define a window but with Analytic Function you can define a window using **_analytic_clause_**

####Anatomy of an Analytics Function

![alt text](http://docs.oracle.com/cd/E11882_01/server.112/e41084/img/analytic_function.gif "analytic function")

E.g. SUM (salary) OVER (ORDER BY last_name, first_name) running_total

* Aggregate function: SUM()
* Column/Expression: salary
* **Over** clause identifies SUM() as an analytic function as opposed to an aggregate function

####Breaking down analytic clause

![alt text](http://docs.oracle.com/cd/E11882_01/server.112/e41084/img/analytic_clause.gif "analytic clause")

Continuing where we left off in the above example:

* Query partition clause: Not used
* ORDER BY clause (_ORDER BY last_name, first_name_) identifies the piece of data this analytic function will be performed over
* windowing_clause: Not used

[Example code from article written by Melanie Caffrey](http://www.oracle.com/technetwork/issue-archive/2013/13-mar/o23sql-1906475.html)

```sql
select last_name, first_name, salary,
SUM (salary) OVER (ORDER BY last_name, first_name) running_total
from employee
order by last_name, first_name;
```
|last_name|first_name|salary|running_total|
|---------|:--------:|:----:|------------:|
|Dovichi|Lori| | |
|Eckhardt|Emily|100000|100000|
|Friedli|Roger|60000|160000|
|James|Betsy|60000|220000|
|Jeffrey|Thomas|300000|520000|
|Michaels|Matthew|70000|590000|
Newton|Donald|80000|670000|
Newton|Frances|75000|745000|
Wong|Theresa|70000|815000|
leblanc|mark|65000|880000|
peterson|michael|90000|970000|

What if my data set has a department ID and i want to see cumulative salary tatal by department ?

I can use **Query Partition Clause** to ensure that the analytic function is applied independently to each department.

```sql
select last_name, first_name, department_id, salary,
SUM (salary) OVER (PARTITION BY department_id ORDER BY last_name, first_name) department_total
from employee
order by department_id, last_name, first_name;
```
|LAST_NAME|FIRST_NAME|DEPARTMENT_ID|SALARY|DEPARTMENT_TOTAL|
|---------|:--------:|:-----------:|:----:|---------------:|
|Dovichi|Lori|10| | |
|Eckhardt|Emily|10|100000|100000|
|Friedli|Roger|10|60000|160000|
|James|Betsy|10|60000|220000|
|Michaels|Matthew|10|70000|290000|
Newton|Donald|10|80000|370000|
|leblanc|mark|20|65000|65000|
|peterson|michael|20|90000|155000|
|Jeffrey|Thomas|30|300000|300000|
|Wong|Theresa|30|70000|370000|
|Newton|Frances| |75000|75000|

The outer **order by** clause does not in any way effect the calculation of department_total, it only sorts the end result according to the column order specified.

But we can manipulate the department total by changing the inner **order by** clause
```sql
select last_name, first_name, department_id, salary,
SUM (salary) OVER (PARTITION BY department_id ORDER BY salary) department_total
from employee
order by department_id,salary, last_name, first_name;
```
|LAST_NAME|FIRST_NAME|DEPARTMENT_ID|SALARY|DEPARTMENT_TOTAL|
|---------|:--------:|:-----------:|:----:|---------------:|
|Friedli|Roger|10|60000|**120000**|
|James|Betsy|10|60000|**120000**|
|Michaels|Matthew|10|70000|190000|
|Newton|Donald|10|80000|270000|
|Eckhardt|Emily|10|100000|370000|
|Dovichi|Lori|10| |**370000**|
|leblanc|mark|20|65000|65000|
|peterson|michael|20|90000|155000|
|Wong|Theresa|30|70000|70000|
|Jeffrey|Thomas|30|300000|370000|
|Newton|Frances| |75000|75000|

You will notice that both Roger and Betsy have same department total, this is because when they are ordered by salary they have exact same numbers for salary and so both are assigned the same department total (which is the sum of their salary).

Also Lori has NULL value for salary and so it is evaluated last and is assigned the highest department_total (running total).

Let's have a look at the windowing clause now, windowing clause lets you control the start & end of window within a particular partition. 

If I have to calculate department_total by adding salary at the current row with salaries of 2 preceding rows, i can use the **ROWS** windowing clause like this:

```sql
select last_name, first_name, department_id, salary,
SUM (salary) OVER (PARTITION BY department_id ORDER BY last_name, first_name ROWS 2 PRECEDING) department_total
from employee
order by department_id, last_name, first_name;
  ```
  
|LAST_NAME|FIRST_NAME|DEPARTMENT_ID|SALARY|DEPARTMENT_TOTAL|
|---------|:--------:|:-----------:|:----:|---------------:|
|Dovichi|Lori|10| | |
|Eckhardt|Emily|10|100000|100000 (0+100000)|
|Friedli|Roger|10|60000|160000 (0+100000+60000)|
|James|Betsy|10|60000|220000 (100000+60000+60000)|
|Michaels|Matthew|10|70000|190000 (60000+60000+70000)|
|Newton|Donald|10|80000|210000|
|leblanc|mark|20|65000|65000|
|peterson|michael|20|90000|155000|
|Jeffrey|Thomas|30|300000|300000|
|Wong|Theresa|30|70000|370000|
|Newton|Frances| |75000|75000|


