Subject|John|Mary
---|---|---
Maths|50|80
Science|85|70

**Write a query to find maximum marks scored by any student in a Subject**

```sql
Create table student
(
  subject varchar(20),
  John int,
  Mary int
  )
  
  INSERT INTO student (subject,John,Mary)
  VALUES
  ('Maths',50,80),('Science',85,70)
```

**Solution # 1**
```sql
SELECT subject, (John+Mary+ABS(John-Mary))/2 as maxmarks
FROM student
```

**Output**

Subject|maxmarks
---|---
Maths|80
Science|85

>**Note:**

>Formula to calculate Max(x,y) = (x + y + ABS(x-y))/2

>Formula to calculate Min(x,y) = (x + y - ABS(x-y))/2

**Solution # 2**
```sql
SELECT subject, MAX(m) FROM (
SELECT subject, John as m
FROM student
UNION 
SELECT subject, Mary
FROM student
) t
GROUP BY subject
```
