---
layout: default
title: Union
parent: Query clauses
grand_parent: Cypher
---

# Database
We will use the database, whose schema and data import commands are given [here](example-database.md):

<img src="../../../img/running-example.png" width="800">

You can import this database by copy pasting the comands on that page. 

*Note: When using the CLI, please modify any multi-line query in the documenation to be in a single line.*

# UNION
`UNION` is the clause where you combine query result A and query result B 
that have the same schema but were produced by different queries into a single result C
by taking their union. Two notes:

- The number of columns and dataType of each column must be identical in A and B but their names can be different.
- UNION will remove all duplicates in the union of A and B. Instead, UNION ALL preserves the duplicates.

For example, the following query returns the age of the follower 
of Zhang and Karissa without duplicate elimination

Query:
```
MATCH (u1:User)-[:Follows]->(u2:User)
WHERE u2.name = 'Zhang'
RETURN u1.age
UNION ALL
MATCH (u3:User)-[:Follows]->(u4:User)
WHERE u4.name = 'Karissa'
RETURN u3.age;
```
Result:

```
------
| 30 |
------
| 40 |
------
| 30 |
------
```

If you changed UNION ALL to UNION in the above query, one of the 30's in the output
would be removed. So the result would be:

```
------
| 30 |
------
| 40 |
------
```
