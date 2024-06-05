# SQL-Murder-Mystery
My solution to the SQL Murder Mystery by NUKnightLab 
https://mystery.knightlab.com/
https://github.com/NUKnightLab/sql-mysteries

A crime has taken place on SQL City and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. 
Start by retrieving the corresponding crime scene report from the police department’s database.

## Exploring Database structure
### Tables

``` sql
SELECT name 
  FROM sqlite_master
 where type = 'table'
```

### Example of structure for the crim_scene_report table
 ```sql
SELECT sql 
  FROM sqlite_master
 where name = 'crime_scene_report'
```
