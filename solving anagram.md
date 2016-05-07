```sql
[Word List](http://www.gutenberg.org/files/3201/files/COMMON.txt)

CREATE TABLE dict
(
	w VARCHAR(50),
    h BIGINT,
    INDEX(w),
    INDEX(h)
);
```
```sql
CREATE TEMPORARY TABLE tmp(w varchar(50),INDEX(w))
```

```sql
LOAD DATA LOCAL INFILE "C:\\Users\\avi\\Documents\\Workplace\\SupportingFiles\\COMMON.txt" /*Source file*/ 
INTO TABLE tmp(w)
```

```sql
UPDATE tmp SET w=REPLACE(REPLACE(LOWER(w),'''',''),'-','');
UPDATE tmp SET w = TRIM(REPLACE(REPLACE(REPLACE(w, '\n', ' '), '\r', ' '), '\t', ' '));

```

```sql
INSERT INTO dict(w)
SELECT DISTINCT w FROM tmp;
```

```sql
SELECT SUM(ORD(SUBSTRING('rat',i,1)))
FROM integers
WHERE i<=LENGTH('rat');
```
Where clause forces you to return only a scalar value, but if you are using an aggregate function it allows you to return more than one valus. Like in this case i<=Length('rat') will return i=1,2,3

Output:
327

**Linear Hash Function**

```sql
UPDATE dict
SET h = 
(
		SELECT SUM(ORD(SUBSTRING(w,i,1)))
		FROM integers
		WHERE i<=LENGTH(w)
)

**Quadratic Hash Function**

```sql
UPDATE dict
SET h = 
(
		SELECT SUM(ORD(SUBSTRING(w,i,1))*ORD(SUBSTRING(w,i,1)))
		FROM integers
		WHERE i<=LENGTH(w)
)
```

