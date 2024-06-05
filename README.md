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
| DATE   | TYPE                                                                                                                                                                                      | DESCRIPTION | CITY |
| -------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| 20180115         | murder       | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".                                                                                                                                                                                          | SQL City         |



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


### Witness interviews

Now that I have both names, I want to query the database (interview table) and find out what the two witnesses said in their interviews. 
I can use their **person_id** info gathered from the previous query to interrogate the **interview** table. 


```sql
SELECT*
FROM interview
WHERE person_id = 14887 -- witness 1
OR person_id = 16371 -- witness 2
;
```

![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/ff11e522-bec2-4857-b13c-5181677da76e)

### Find your suspect

We now know that both witnesses saw one man running away from the murder scene. 

We also know that:
-  This man had a "**Get Fit Now Gym**" bag. 
-  The number on the bag started with a "**48Z**".
-  Only gold members carry those bags.
-  The man's plate included "**H42W**".

To find a potential suspect, we need to gather all the new info from the witnesses' interviews and cross-reference them in the same table. I will create a new table by joining data in tables person(id), get_fit_now_member (membership_status) and drivers_license tables (plate_number).
I also want to simplify the names, so I will use **ALIAS** in some cases. 

![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/8e2bc6a7-0054-470f-b472-f1eb61ee14a4)

### Finding the killer!

We know that a man named Jeremy Bowers could be our culprit, but to be sure we need to prove he was on the crime scene at the time of the murder.

To do this, we compare the gym check-in and check-out times of our suspect with Annabel Miller for the day of the murder (9th of January 2018) and see if they are compatible: Annabel and the suspect were both at the same gym the day of the murder).

### Finding the killer!

We know that a man named Jeremy Bowers could be our culprit, but to be sure we need to prove he was on the crime scene at the time of the murder.

To do this, we compare the gym check-in and check-out times of our suspect with Annabel Miller for the day of the murder (9th of January 2018) and see if they are compatible: Annabel and the suspect were both at the same gym the day of the murder).

```sql 

SELECT check_in_time, check_out_time, name
FROM get_fit_now_check_in AS gfnc
JOIN get_fit_now_member AS gfnm
ON gfnm.id = gfnc.membership_id
WHERE  check_in_date = '20180109'
AND name LIKE "Annabel%" 
OR name LIKE "Jeremy%"
;

```
![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/94e2217f-9ff9-4be1-a980-6c568f3e0474)

Annabel and Jeremy checked out at the same time (5:00 pm) and this put Jeremy at the crime scene at the time of the murder.

Jeremy Bowers is indeed the murderer! 

![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/7585a70d-a35f-4103-b82d-8525e0119ce5)


### One step further... to find the REAL culprit. 

We want to read Jeremy's statement, to find out if there is more to this case. 

To make this easier, I join the **interview** table and the **person** table together.

```sql

SELECT person_id, transcript, id, name
FROM interview
JOIN person 
 ON interview.person_id = person.id
WHERE  
 name LIKE "Jeremy Bowers"
;  

```

![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/486a4140-f00d-4420-9880-7fe3aa87cb9a)


By reading Jeremy's statement, a woman hired Jeremy to commit the murder in her place that day.

Jeremy says that: 
- He was hired by a **woman with a lot of money**. He doesn't know her name.
- He knows she's around **5'5" (65") or 5'7" (67")**. She has **red hair** and she drives a **Tesla Model S**.
-  He knows that she **attended the SQL Symphony Concert 3 times in December 2017**.

I then join together several tables (drivers_license, person, facebook_event_checkin, income) to check all the info (in bold) from Jeremy's interview and find out the real culprit. 

```sql

SELECT dl.id, height, hair_color, gender, car_model, event_name, fc.date, p.name, p.ssn, annual_income
FROM drivers_license AS dl
JOIN person AS p
ON p.license_id = dl.id
JOIN facebook_event_checkin AS fc
ON fc.person_id = p.id
JOIN income AS inc
ON inc.ssn = p.ssn 
WHERE gender = "female"
AND hair_color = "red"
AND height BETWEEN 65 AND 67
AND car_model = "Model S"
;  

```

![image](https://github.com/Camilla82/SQL-Murder-Mystery/assets/126681504/18e7e716-ac90-4920-b5b1-fd35b05f8280)

The woman behind the murder is **Miranda Priestly**!!
