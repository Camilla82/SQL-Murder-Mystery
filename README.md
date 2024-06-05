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

| ID             | NAME       | LICENCE_ID     | ADDRESS_NUMBER      | ADDRESS_STREET_NAME | SSN |
| ----- | -------------- | ---------- | -------------- | ------------------- | --------- |
| 14887      | Morty Schapiro               | 118009               | 4919                         | Northwestern Dr                        | 111564949          |


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

| ID             | NAME       | LICENCE_ID     | ADDRESS_NUMBER      | ADDRESS_STREET_NAME | SSN |
| ----- | -------------- | ---------- | -------------- | ------------------- | --------- |
| 16371      | Annabel Miller               | 490173               | 103                          | Franklin Ave                           | 318771143          |


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

| PERSON_ID                                                                                                                                                                                                                       | TRANSCRIPT |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 14887              | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".                                                                                                                                                                                                                                |
| 16371              | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                                                                                                                                                                                                                                                                                          |


### Find your suspect

We now know that both witnesses saw one man running away from the murder scene. 

We also know that:
-  This man had a "**Get Fit Now Gym**" bag. 
-  The number on the bag started with a "**48Z**".
-  Only gold members carry those bags.
-  The man's plate included "**H42W**".

To find a potential suspect, we need to gather all the new info from the witnesses' interviews and cross-reference them in the same table. I will create a new table by joining data in tables person(id), get_fit_now_member (membership_status) and drivers_license tables (plate_number).
I also want to simplify the names, so I will use **ALIAS** in some cases. 

```sql 

SELECT person.id, gfnm.name, person.license_id, gfnm.membership_status, dl.plate_number 
FROM person
JOIN get_fit_now_member AS gfnm
ON  person.id = gfnm.person_id
JOIN drivers_license as dl
ON person.license_id = dl.id 
WHERE gfnm.membership_status = "gold"
AND gfnm.id LIKE "48Z%"
AND dl.plate_number LIKE "%H42W%"
;

```

| ID            | NAME       | LICENCE_ID        | MEMBERSHIP_STATUS | PLATE_NUMBER |
| ----- | ------------- | ---------- | ----------------- | ------------ |
| 67318      | Jeremy Bowers              | 423327               | gold                               | 0H42W2                   |


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

| CHECK_IN_TIME  | CHECK_OUT_TIME | NAME |
| ------------- | -------------- | -------------- |
| 1530                       | 1700                         | Jeremy Bowers                |
| 1600                       | 1700                         | Annabel Miller               |

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

| PERSON_ID                                                                                                                                                                                                                                        | TRANSCRIPT | ID            | NAME |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----- | ------------- |
| 67318              | I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.                                                                                                                                                                                                                                                 | 67318      | Jeremy Bowers              |


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

| ID     | HEIGHT     | HAIR_COLOR | GENDER    | CAR_MODEL            | EVENT_NAME | DATE             | NAME          | ANNUAL INCOME |
| ------- | ------ | ---------- | ------ | --------- | -------------------- | -------- | ---------------- | ------------- |
| 202298	        | 66           | red                  | female       | Model S            | SQL Symphony Concert                     | 20171206         | Miranda Priestly                 | 310000                     |
| 202298	        | 66           | red                  | female       | Model S            | SQL Symphony Concert                     | 20171212         | Miranda Priestly                 | 310000                     |
| 202298	        | 66           | red                  | female       | Model S            | SQL Symphony Concert                     | 20171212         | Miranda Priestly                 | 310000                     |


The woman behind the murder is **Miranda Priestly**!!
