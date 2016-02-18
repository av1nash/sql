# Analytic Functions
[Oracle Analytic Functions Reference Page](http://docs.oracle.com/cd/E11882_01/server.112/e41084/functions004.htm#SQLRF06174)
```
Analytic Functions or Window Functions compute an aggregate value based on a group of rows.
```
The group of rows is called a **window** and that is how they are different from just an aggregate function. A simple aggregate function doesn't let you define a window but with Analytic Function you can define a window using **_analytic_clause_**

####Anatomy of an Analytics Function

![alt text](http://docs.oracle.com/cd/E11882_01/server.112/e41084/img/analytic_function.gif "analytic function")

E.g. SUM (salary) OVER (ORDER BY last_name, first_name) running_total

Aggregate function: SUM()
Column/Expression: salary

####Breaking down analytic clause

![alt text](http://docs.oracle.com/cd/E11882_01/server.112/e41084/img/analytic_clause.gif "analytic clause")

Continuing where we left off in the above example:

Query partition clause: Not used
order by clause: ORDER BY last_name, first_name
windowing_clause: Not used

[Example code from article written by Melanie Caffrey](http://www.oracle.com/technetwork/issue-archive/2013/13-mar/o23sql-1906475.html)
