# SQL-data-cleaning

This is an educational project on data cleaning and preparation using SQL. The original database in CSV format is located in the file club_member_info.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset.

## View data
View 10 first rows
```sql
SELECT *
FROM club_member_info cmi 
LIMIT 10;
```
The result:
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

## Copy the table 
### Creat a new table for cleaning
```sql
-- club_member_info definition

CREATE TABLE club_member_info_cleaned (
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone VARCHAR(50),
	full_address VARCHAR(50),
	job_title VARCHAR(50),
	membership_date VARCHAR(50)
);
```
### Copy all values from original table
```sql
INSERT INTO club_member_info_cleaned 
SELECT * FROM club_member_info;
```
## Clean data
### Inconsistent letter case in full_name
Eleminate excess space
```sql
UPDATE club_member_info_cleaned
SET full_name = TRIM(full_name);
```
Capitalize names in full_name column
```sql
UPDATE club_member_info_cleaned
SET full_name = UPPER(full_name);
```
Result:
|full_name|
|---------|
|ADDIE LUSH|
|ROCK CRADICK|
|SYDEL SHARVELL|
|CONSTANTIN DE LA CRUZ|
|GAYLOR REDHOLE|
|WANDA DEL MAR|
|JOANN KENEALY|
|JOETE CUDIFF|
|MENDIE ALEXANDRESCU|
|FEY KLOSS|

### Age out of realistic range
Process the data with age > 100 & age IS NOT NULL
```sql
UPDATE club_member_info_cleaned
SET age = SUBSTR(age, 1, 2)
WHERE age > 100 AND age != '';
```
Replace empty age values with Mean values of age column
```sql
UPDATE club_member_info_cleaned 
SET age = (
	SELECT ROUND(AVG(age),0) FROM club_member_info_cleaned cmic 
)
WHERE age = '';
```

### Process the martial_status 
Problem: Empty values & Spelling error "divored"
```sql
SELECT DISTINCT(martial_status) FROM club_member_info_cleaned cmic
```
|martial_status|
|--------------|
|married|
|divorced|
||
|single|
|divored|

Replace empty values with 'Unknown'
```sql
UPDATE club_member_info_cleaned 
SET martial_status = 'Unknown'
WHERE martial_status = '';
```
Replace 'divored' with 'divorced'
```sql
UPDATE club_member_info_cleaned
SET martial_status = 'divorced'
WHERE martial_status = 'divored';
```
### Delete duplicates
```sql
DELETE FROM club_member_info_cleaned  
WHERE ROWID IN (
	SELECT cmic.ROWID FROM club_member_info_cleaned cmic 
	JOIN 
		(SELECT email, MIN(ROWID) AS min_id FROM club_member_info_cleaned 
		GROUP BY email) AS cmicc
	ON cmic.email = cmicc.email 
	WHERE cmic.ROWID > cmicc.min_id
);
```
