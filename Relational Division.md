I was reading this [blog](https://blog.jooq.org/2012/03/30/advanced-sql-relational-division-in-jooq/) which explains how you can use the concept of Relational Division in SQL.
This invloves use of doubly-nested select statements using anti-joins.
I always had problems understanding nested SQL queries, especially if they are using anti-joins (NOT IN or NOT EXISTS).
This is my attempt to record my understanding of how to decipher such queries and more importantly, how to write one from scratch.

Before we jump into Relational Division, let us look at a simple nested sql query.
EXISTS and NOT EXISTS are examples of correlated subqueries, in which the outer query's first row is selected and then linked with inner query's table, if it is linked and satisfies any other condition in the inner query one or more rows will be returned.
If an EXISTS clause is used that will translate to include this row from the outher query in your result set and opposite for NOT EXISTS.

In the query below we want to select all the rows from Contact table if custid is also present in customer table and customer state is 'FL'

```sql
SELECT * FROM Contact c
WHERE EXISTS 
(
  SELECT * FROM Customer cu
  WHERE State='FL'
  AND c.custid = cu.custid
);
```
1. custid column links the Contact table to the Customer table.
2. SQL looks at the first record in the Contact table, finds the row in the Customer table that has the same custid, and checks that row’s State field.
3. If cu.State = ‘FL’, the current Contact row is added to the result table.
4. The next Contact record is then processed in the same way, and so on, until the entire Contact table has been processed.

If our requirement would have been to select all Contact where such that no Customer from State 'FL' is selected, we would use NOT EXISTS instead.

Now lets take a look at the Relational Division in SQL, let us use the same example which the blog post above used (which in turn is a wikipedia example)

We have two tables, Completed and DBProject

**Completed**

|Student|Task|
--- | --- |
Fred|Database1
Fred|Database2
Fred|Compiler1
Eugene|Database1
Eugene|Compiler1
Sarah|Database1
Sarah|Database2

**DBProject**

| Task |
---
|Database1|
|Database2|

Our aim is to write a query to find out Students who have completed all the tasks listed under Task table.
Without the use of Relational Division, you can write this query to find the answer - 

```sql
SELECT c1.Student
FROM Completed c1, DBProject db
WHERE c1.TASK=db.TASK
GROUP BY c1.Student
HAVING COUNT(c1.task) = (SELECT count(Task) from DBProject);
```

And use below mentined query which implements the concept of Relational Division to find the right answer
```sql
SELECT DISTINCT Student From Completed c1
WHERE NOT EXISTS
(
  SELECT * FROM DBProject WHERE NOT Exists
    (
      (SELECT * FROM Completed c2 WHERE c2.Student=c1.Student and c2.Task=DBProject.Task)
    )
)
```

Let's diseect the second query since first one is pretty self explanatory
SQL will look at the first record in Completed c1 table and try to link/join that with Completed c2 table on Student. Let us say that the first Student was picked up as 'Fred'. When joined with Completed c2 table we should get three rows:
|Student|Task|
--- | --- |
Fred|Database1
Fred|Database2
Fred|Compiler1

Maybe we can write the entire query for this value of Student ('Fred') like this:
```sql
SELECT * FROM DBProject WHERE NOT EXISTS
(
  SELECT * FROM Completed
  WHERE Student = 'Fred'
  AND Completed.Task=DBProject.Task
)
```
Above query will not return anything, because for all the tasks in DBProject there is a matching row in Completed c2 for Student='Fred'

Maybe to further make our point clearer, we can reqrite the query to replace NOT EXISTS with EXISTS
```sql
SELECT * FROM DBProject WHERE EXISTS
(
  SELECT * FROM Completed
  WHERE Student = 'Fred'
  AND Completed.Task=DBProject.Task
)
```
Output of this query is:

| Task |
---
|Database1|
|Database2|

Can you guess what will be the result if you replace Fred with Eugene in this query. In plain English you will be asking the question " Hey ! is there any Task in DBProject which Eugene has not completed ?"
```sql
SELECT * FROM DBProject WHERE NOT EXISTS
(
  SELECT * FROM Completed
  WHERE Student = 'Eugene'
  AND Completed.Task=DBProject.Task
)
```
And you know the answer to that one - "Yes, Eugene has not completed Task - Database2"

| Task |
---
|Database2|

That is why Eugene will not be included in the Final Result set.
