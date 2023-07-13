The detective has provided with some information about the missing file, the goal at this stage is to retrieve the corresponding crime scene report from  the police department’s database.
To do this, a couple of queries were written to query the database in order to get the desired data from the crime_scene_report table which houses data about crimes.

```
Select * 
From crime_scene_report
Where type = 'murder' and date = 20180115 and city = 'SQL City';
```

