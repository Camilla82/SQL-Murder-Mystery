# SQL-Murder-Mystery
My solution to the SQL Murder Mystery by NUKnightLab 

> https://mystery.knightlab.com/
>
> https://github.com/NUKnightLab/sql-mysteries

## Task

> A crime has taken place on SQL City and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​.
> 
> Start by retrieving the corresponding crime scene report from the police department’s database. 


The entity relationship diagram (ERD) showing the different tables in the database and the relationship between them is also provided by the challenge (see below):

![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/bad1658e-d5be-4a5b-bf83-2392b419298f)



### Exploring Database structure

``` sql
SELECT name 
  FROM sqlite_master
 where type = 'table'
```

### Table structure - example
 ```sql
SELECT sql 
  FROM sqlite_master
 where name = 'crime_scene_report'
```


## Solution
