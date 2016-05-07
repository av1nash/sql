You can download a word list from a site like gutenberg which will act as a dictionary of words. This is a link to one such [Word List](http://www.gutenberg.org/files/3201/files/COMMON.txt)

Basic idea is that anagrams (like rat - tar) will have the same set of alphabets so if we apply some hash function on a word and calculate a hash value, then all the anagrams of a word will have the same hash value and you can filter your table by the value to find it.

**Script # 1 - Create dict table to hold the words and their hash values**
```sql
CREATE TABLE dict
(
    w VARCHAR(50),
    h BIGINT,
    INDEX(w),
    INDEX(h)
);
```

**Script # 2 - Create a temporary table where we will load the data from the word list file and apply some transformations before finally loading it into the dict table**

```sql
CREATE TEMPORARY TABLE tmp(w varchar(50),INDEX(w))
```

```sql
LOAD DATA LOCAL INFILE "C:\\Users\\avi\\Documents\\Workplace\\SupportingFiles\\COMMON.txt" /*Source file*/ 
INTO TABLE tmp(w)
```

**Script # 3 - Update tmp table to make all letters small case and replace semi colons, new line, carriage return, tab and hyphen with blank**

```sql
UPDATE tmp SET w=REPLACE(REPLACE(LOWER(w),'''',''),'-','');
UPDATE tmp SET w = TRIM(REPLACE(REPLACE(REPLACE(w, '\n', ' '), '\r', ' '), '\t', ' '));
```

**Script # 4 - To make sure we are not loading duplicates in our dict table use DISTINCT select from tmp table**

```sql
INSERT INTO dict(w)
SELECT DISTINCT w FROM tmp;
```

**Concept Learned # 1**
Where clause forces you to return only a scalar value, but if you are using an aggregate function it allows you to return more than one valus. Like in this case i<=Length('rat') will return i=1,2,3

```sql
SELECT SUM(ORD(SUBSTRING('rat',i,1)))
FROM integers
WHERE i<=LENGTH('rat');
```
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
```
This gives us a lot of false positives, lets try another hash function

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

**Exponential Hash Function**
BIGINT in MySQL is of 8 bytes (8*8 = 64 bits)
2^63 ..... 2^2,2^1,2^0
So you can say that there are slots from 0 - 63 where you can put one alphabet, lets say that we put a at slot 0 and b in slot 1 and so on such that slot 25 is occupied by z
If we apply this hash function to a word say "abc" we get a value 6 and if the word is bc, well we again get 6. This is not very accurate.
What can we do to make it more accurate, problem is that position 0 doesn't hold any value so lets shift everything to the left, in practice we can do that by using bit shift operator **<<** 

```sql
UPDATE dict
SET h = 
(
		SELECT SUM(1<<(ORD(SUBSTRING(w,i,1))-97)*2)
		FROM integers
		WHERE i<=LENGTH(w)
);
```

One thing that might be bothering you (as it did to me) is why -97 and x2, well ordinal value of a is 97 if I use bitwise operator 1<<97 i will get 0 because my BIGINT i has only 63 slots, if i shift something 97 times all i will get is 0s.
And we are multiplying by 2 so that alphabets occupy slots with a gap of 1, so that when shifting occur they all represent unique slots, for e.g. a will occupy slot 0, b will occupy slot 2 and z will occupy slot 50. In other wors you can say that each alphabet is represented by 2 bits.


