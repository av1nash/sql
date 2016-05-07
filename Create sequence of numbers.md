Following script can be used to insert Sequential Numbers within a limit (1-64) in an empty table

```sql
CREATE TABLE integers
(
	i BIGINT
);
```

To insert numbers from 1 to 64 in TABLE integers

```sql
DELIMITER $$
CREATE PROCEDURE dowhile()
BEGIN
  DECLARE v1 INT DEFAULT 1 ;

  WHILE v1 < 65 DO
    INSERT INTO integers
    Values(v1);
    SET v1 = v1 + 1;
  END WHILE;
END;
```

```sql
CALL dowhile()
```
