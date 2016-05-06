This is the Logical query processing order of a SQL query engine, note that query optimizer might choose a different order than this when running the query. 
But for most practical purposes of coding, you can rely on this order.

1. FROM
2. ON
3. JOIN
4. WHERE
5. GROUP BY
6. WITH CUBE/ROLLUP
7. HAVING
8. SELECT
9. DISTINCT
10. ORDER BY
11. TOP
12. OFFSET/FETCH

**For Example:** Since ORDER BY clause comes after SELECT CLAUSE, you can use a calculation like *count(*) as cnt* in your ORDER BY clause *ORDER BY cnt*
