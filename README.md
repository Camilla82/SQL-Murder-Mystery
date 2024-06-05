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


The challenge also suggests you start with these two queries (see below): 

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


## My solution

### Murder report 

Considering the information I have been given by the challenge (*crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City* ), I will start by interrogating the database about the **murder report**: 


``` sql
SELECT * 
FROM crime_scene_report
WHERE type = 'murder'
AND city = 'SQL City'
AND date = '20180115'
;

```
![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/855f4e39-dced-4cce-b651-4fe50b735166)


We found out that we have two witnesses:
 1) A man living **"on the last house on Northwestern Dr"**.
 2) A woman named **"Annabel"** living on **"Franklin Ave"**.

### Find witnesses personal details

#### First witness

I want to find the info for the first witness' personal details. 
I write a query to interrogate the **"person"** table for all entries associated to the **"Franklin Ave"** address and I order them by their address number (descendant order) so to find the last number associated for that specific address. I then put the limit to one result.  

``` sql
SELECT * 
FROM person
WHERE address_street_name = 'Northwestern Dr' 
ORDER BY address_number DESC
limit 1
;
```
![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/670e12c8-8505-443b-b9af-6293ba70a12b)

The name of the person living on the last house on Northwestern Dr is **"Morty Schapiro"**

#### Second witness

I then want to find the second witness' personal idetails.
I write a query to interrogate the **"person"** table for all names like **"Annabel"** associated to the **"Franklin Ave"** address. 

``` sql
SELECT*
FROM person
WHERE name LIKE "Annabel%"
AND address_street_name = "Franklin Ave"
;
```
![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/ec6fd277-2bda-4cd7-b6aa-6ae434db7fe7)







