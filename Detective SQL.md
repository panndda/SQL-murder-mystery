## STEP 1
The goal at this stage is to retrieve the corresponding crime scene report from the police department’s database.
To do this, a couple of queries were written to query the database in order to get the desired data from the crime_scene_report table which houses data about crimes.

```sql
Select * 
From crime_scene_report
Where type = 'murder' and date = 20180115 and city = 'SQL City';
```
![1](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/1.PNG)

The above query returns the date of the murder, type of murder, city where it occurred and a brief description of the crime scene picked from a security footage. The security footage revealed that there were 2 witnesses at the scene; the first witness lives at the last house on "Northwestern Dr" while the second witness, named Annabel, lives somewhere on "Franklin Ave". 
> Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". >

The next step will be to try to locate the two witnesses identified.


## STEP 2
With the little information about Annabel and the other witness, I would query the "person" table, the "person" table houses basic data (id, name, address and so on) about people. Below is the query written to get data from the table; 
```sql
SELECT * 
FROM person
WHERE (name like '%Annabel%' and address_street_name ='Franklin Ave') or 
		(address_number = (select max(address_number) from person) and address_street_name = 'Northwestern Dr')
```
![2](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/2.PNG)

Annabel and Morty will be invited for an interview to get some info from them. 

## STEP 3: 
At the interview, Annabel said;
> I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

Morty said;
> I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

The query used to retrieve details about the interview session with Annabel and Morty; 
```sql
Select p.name,i.person_id, i.transcript
From interview as i
join person as p
on i.person_id=p.id
Where person_id in (14887,16371)  
```
![3a](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/3a.PNG)

![3b](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/3b.PNG)

The next point of call will be to use the information obtained from both witnesses to trace the murderer, the following info were obtained;
* Annabel recognized The murderer from gym, she was there on 9th of january
* The murderer had a "Get Fit Now Gym" bag
* The membership number started with "48Z"
* Only Gold members had those bags
* He is a Male
* He got into a Car with a plate that included "H42W"
From the points above, the killer is most probably a gym member.

## STEP 4: Finding the killer
* The first point of call at this stage was to check if Annabel was truly a member of the gym, the query to check this is;	
```sql
Select * 
From get_fit_now_member 
Where person_id =16371;
```
![4](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/4.PNG)

The above query returned with some data which confirmed that Annabel was truly a member of the gym with membership number “90081”

* 
* Now the next action is to check for the time when Annabel was at the gym on the 9th of January.
```sql
Select *
From get_fit_now_check_in
Where check_in_date=20180109 and membership_id=90081;
```
![5](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/5.PNG)

The above query ad output affirmed that Annabel checked in at 16:00 and checked out at 17:00.

* 
* The next task is to get the list of people that were in the gym at the same time time with Annabel,
```sql
Select *, case when check_in_time >= 1600 or check_out_time >= 1600 then 'in'
	else 'out' end as in_out_gym
From get_fit_now_check_in
Where check_in_date=20180109 and in_out_gym='in';
```
![6](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/6.PNG)

The above query and result display information about the people that were checked into the gym after 1600 and the people that checked out after 1600. The query returned 3 rows of data, Annabels’ data inclusive.

# STEP 5: 
person_id and names were retrieved from the get_fit_member table to know the identity of the 2 suspects . The query below was used to do that;
```sql
Select * 
From get_fit_now_member 
Where id ='48Z55' OR id='48Z7A';
```
Their was names and membership_status were also retrieved
![7](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/7.PNG)

At this point, the results obtained so far show that both of the suspects(Jeremy Bowers and Joe Germuska) ticked all but one of the boxes as regards the information supplied by Annabel and Morth;
* Thet were both at the gym at the same time with Annabel
* Their membership number started with "48Z"
* They were both Gold members
* They are both Male
  
Now, we try to also factcheck the last information about the car plate.

# Step 6:
The syntax below queries the drivers_license table to know the owner of the car plate.
```sql
SELECT *
from drivers_license
where  plate_number like '%H42W%'
```
![8](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/8.PNG)

The query returned three rows(1 Female and 2 Males), so we are concerned about the 2 males, now we have 2 ids for people that had "H42W" as part of their number plate.
Now lets query the "person" table to know the name of the 2 people with those ids.
```sql
Select * 
From person 
Where license_id  in (423327,664760)
```
![9](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/9.PNG)

BOOOM, Now we have the names of the 2 plate number owners(Jeremy Bowers , Tushar Chandra)


Now we have the name of the murderer as he is the only one that ticked all the boxes as specified by Annabel and Morty.

# Step 7:
Now the suspect will be brought in for interrogation, the query below returns the interview details with Jeremy Bowers
```sql
Select * 
From interview
Where person_id=67318
```
At this stage, the query returned the confession of one of the suspects with person_id (67318) where he said;
> I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017. She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

![10](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/10.PNG)

## Step 8:
Now, he has supplied us with information about person that sent him. They are;
1. Tesla
2. Model S
3. 65” OR 67”
4. Red Hair
5. Female
6. Rich
7. Attended SQL Symphony 3 times in December in 2017.
  
We are going to query the drivers_license table to get information about the suspect that sent Jeremy, the query below would do that
```sql
SELECT * 
FROM drivers_license 
WHERE hair_color='red' and car_make='Tesla' AND gender='female'
 and car_model='Model S' AND height IN (65,67)
```
![11](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/11.PNG)

This returns a only a row of data, it satisfies criteria (1 - 5) above


```sql
Select * 
From person 
Where license_id=918773
``` 
The query above returns the person_id and ssn in order to query the events table and income table)
This returns the person_id, name  and ssn of the suspect

![12](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/12.PNG)


* 
* The syntax below queries the facebook table to get info about the suspect's attendance
```sql
SELECT * 
FROM facebook_event_checkin 
WHERE person_id=78881 AND event_name='SQL Symphony Concert' 
Order by person_id
```
This query is to check if the criteria (7) above is satisfied, but the criteria isn't satisfied as the query returns a null table which means Red Korb with the person_id 78881 did not attend the SQL concert.

![13](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/13.PNG)

* AnnuAL Income
```sql
Select *, round((Select avg(annual_income)  From income)) as average_income
from income 
Where ssn=961388910
```

![14](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/14.PNG)

Then the condition  (6) was also satisfied as Red Korb was earning than the average salary range.


## Step 9: Further Analysis
A further analysis was done to uncover the suspect that sent Jeremy as the condition (7) wasn't met, the condition (3) was modified to contain (65,66,67) as no one is expected to correctly ascertain a person’s height by just mere stare.
```sql
SELECT * 
FROM drivers_license 
WHERE hair_color='red' and car_make='Tesla' AND gender='female'
 and car_model='Model S' AND height IN (65,66,67)
```

![15](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/15.PNG)

The above query returns 3 suspects as they all satisfy criteria (1 - 5)


```sql
Select * 
From person 
Where license_id=918773 or license_id=291182 or license_id=202298
```
![16](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/16.PNG)

This returns the person_id, name and ssn of the suspects.

* This query is used to check the attendance of each of suspects at the facebook event
```sql
SELECT name,ssn,address_number ||' '||address_street_name,person_id, COUNT(*) as attendance 
FROM facebook_event_checkin as f
left join person as p
on p.id=f.person_id
WHERE person_id=78881 or person_id=90700 or person_id=99716 and event_name='SQL Symphony Concert' 
GROUP by 1
Order by person_id
```

![17](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/17.PNG)

This returns only 1 person_id (99716) which means the this suspect attended the SQL concert 3 times (CONDITION 7). The name of the suspect is Miranda Priestly of  1883 Golden Ave with ssn(987756388)


```sql
Select *, round((Select avg(annual_income) From income)) as average_income
From income 
Where ssn in (961388910,337169072,987756388)
```
![18](https://github.com/panndda/SQL-murder-mystery/blob/main/New%20folder/18.PNG)

Lastly, the last condition (6) was also satisfied by only 2 suspects(Miranda Priestly and Red Korb ) as they earn waayyy higher than the average. So only Miranda Priestly ticked all the boxes.

# CONCLUSION
Based on the analysis so far, it can concluded that the murderer is Jeremy Bowers with the id(67318) of 530 Washington Pl, Apt 3A with ssn(871539279).  Miranda Priestly of  1883 Golden Ave with person_id (99716) hired Jeremy to kill.




