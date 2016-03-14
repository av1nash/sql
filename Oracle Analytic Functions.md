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

Windowing clause provides following two views of data:

1. Anchored view - When you use only an ORDER BY clause, it begins with the first row of the partition and ends with the current row being processed.

2. Sliding view - When the value being calculated (e.g. department_total) can change depending on how the data is sorted within each partition.

Let's look at the **RANGE** windowing clause which provides a sliding view of the data

```sql
select last_name, first_name, department_id, hire_date, salary,
SUM (salary) OVER (PARTITION BY department_id ORDER BY hire_date RANGE 90 PRECEDING) department_total
from employee
order by department_id, hire_date;
```

In above query we want to sort our partition by hire_date and apply our aggregate function SUM() on a window of hire dates which are in the range of **90** days preceding the hire_dt on current row.

|LAST_NAME|FIRST_NAME|DEPARTMENT_ID|HIRE_DATE|SALARY|DEPARTMENT_TOTAL|
|---------|:--------:|:-----------:|:-------:|:----:|---------------:|
|Eckhardt|Emily|10|07-JUL-04|100000|100000|
|Newton|Donald|10|24-SEP-06|80000|80000|
|James|Betsy|10|16-MAY-07|60000|190000|
|Friedli|Roger|10 |16-MAY-07|60000|190000|
|Michaels|Matthew|10 |16-MAY-07|70000|190000|
|Dovichi|Lori|10 |07-JUL-11| | |
|peterson|michael|20 |03-NOV-08|90000|90000|
|leblanc|mark|20 |06-MAR-09|65000|65000|
|Jeffrey|Thomas|30 |27-FEB-10|300000|370000|
|Wong|Theresa|30 |27-FEB-10|70000|370000|
|Newton|Frances| |14-SEP-05|75000|75000|

```
Note: I have sort of compiled this text from Oracle's reference page and Oracle magazine article for my own personal use (so that i can quickly refresh my memory about a concept). Hopefully you (the reader) will also find it useful. I have tried to include the links to the reference page and give due credits wherever applicable. If i missed out something, please inform me and I will sort it out.
```
