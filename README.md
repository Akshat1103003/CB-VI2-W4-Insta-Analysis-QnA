### Q1. How many unique post types are found in the 'fact_content' table ?
Query : 
```sql
select distinct post_type  from fact_content ;
```
![image](https://github.com/user-attachments/assets/56809f75-075a-4040-909c-25401c81a51c)

### Q2.What are the highest and lowest recorded impressions for each post type ? 
Query :
```sql
select 
		post_type,
        max(fc.impressions) as MaximumImpressions ,
        min(fc.impressions) as MinimumImpressions 
        from fact_content fc 
        group by post_type ;
```
![image](https://github.com/user-attachments/assets/5ba11804-8b86-44e1-bbc9-86065742ada3)

### Q3. Filter all the posts that were published on a weekend in the month of March and April and export them to a separate csv file.
Query :
```sql
select fc.*,dd.weekday_or_weekend
from fact_content fc
    join dim_dates dd
        on fc.date = dd.date 
	where month(fc.date) between 3 and 4 
	    and
	dd.weekday_or_weekend ="Weekend" ;
```
![image](https://github.com/user-attachments/assets/52eadd42-0611-425a-93e0-76fde528d0cf)


### Q4. Create a report to get the statistics for the account. The final output includes the following fields : 
• month_name  
• total_profile_visits  
• total_new_followers  
 Query :
 ```sql
select 
	monthname(fa.date) as MonthName,
	sum(profile_visits) as total_profile_visits,
    sum(new_followers) as total_new_followers 
    from fact_account fa 
group by MonthName ;
```
![image](https://github.com/user-attachments/assets/01627b7e-517e-41e0-88c8-6383fbada97d)

### Q5. Write a CTE that calculates the total number of 'likes’ for each 'post_category' during the month of 'July' and subsequently, arrange the 'post_category' values in descending order according to their total likes .
Query :
```sql
with cte_1 as (
select 
	post_category,
    sum(fc.likes)  as total_likes 
from fact_content fc 
where monthname(fc.date)="July"
group by post_category )
select post_category ,total_likes  from cte_1
cte_1 order by total_likes ;
```
![image](https://github.com/user-attachments/assets/a4e99383-0909-4fc7-90df-3d43674232e8)

### Q6. Create a report that displays the unique post_category names alongside their respective counts for each month. The output should have three columns :
• month_name   
• post_category_names   
• post_category_count  
Query :
```sql
select 
	group_concat(distinct fc.post_category order by fc.post_category asc separator "," ) as Categories,
    monthname(fc.date) as month_name
from fact_content fc 
group by month_name ;
```

![image](https://github.com/user-attachments/assets/999def14-26c0-44d1-bdfa-79b502f01cb0)

### Q7. What is the percentage breakdown of total reach by post type? The final output includes the following fields:
• post_type  
• total_reach  
• reach_percentage  
Query :
```sql
with cte_1 as (
	select fc.post_type,
	sum(fc.reach) as total_reach ,
	monthname(fc.date) as month_name
    from fact_content fc
    group by month_name,fc.post_type
    )
select *,total_reach*100 / sum(total_reach) over (partition by post_type) as reach_perc 
from cte_1 ;
```
![image](https://github.com/user-attachments/assets/576adbcc-b3cb-4dc1-b5fa-ddbc6af3bf9e)

### Q8. Create a report that includes the quarter, total comments, and total saves recorded for each post category. Assign the following quarter :
groupings :  
(January, February, March) → “Q1”  
(April, May, June) → “Q2”  
(July, August, September) → “Q3”  
The final output columns should consist of:  
• post_category  
• quarter  
• total_comments  
• total_saves  
Query :
```sql
WITH cte_1 AS (
    SELECT
        fc.post_category,
        fc.date,
        MONTHNAME(fc.date) AS month_name,
        MONTH(fc.date) AS month_num,  
        fc.comments,
        fc.saves,
        CASE  
            WHEN MONTH(fc.date) BETWEEN 1 AND 3 THEN 'Q1'
            WHEN MONTH(fc.date) BETWEEN 4 AND 6 THEN 'Q2'
            WHEN MONTH(fc.date) BETWEEN 7 AND 9 THEN 'Q3'
            WHEN MONTH(fc.date) BETWEEN 10 AND 12 THEN 'Q4'
            ELSE 'Error'
        END AS quarter
    FROM fact_content fc
)
SELECT 
    post_category,
    month_name, 
    month_num,  
    quarter,
    SUM(comments) AS total_comments, 
    SUM(saves) AS total_saves
FROM cte_1
GROUP BY post_category, month_name, month_num, quarter
ORDER BY month_num, post_category;
```
![image](https://github.com/user-attachments/assets/066286f7-af46-4999-900f-49adb78c1625)


### Q9. List the top three dates in each month with the highest number of new followers. The final output should include the following columns :
• month  
• date  
• new_followers  
Query :  
```sql
with cte_1 as (
select 
	month(fa.date) as month ,
    fa.date ,fa.new_followers ,
	rank() over (partition by month(fa.date) order by fa.new_followers desc) as nf_rank
    from fact_account fa )
select month ,date ,new_followers ,nf_rank 
from cte_1
where nf_rank <=3
order by month(date) ,nf_rank ,date ;
```
![image](https://github.com/user-attachments/assets/a5b51346-66a6-4eac-b427-753b547b5626)

### Q10. Create a stored procedure that takes the 'Week_no' as input and generates a report displaying the total shares for each 'Post_type'. The output of the procedure should consist of two columns:
• post_type  
• total_shares  
Query :
```sql
with cte_1 as ( 
select sum(fc.shares) as total_shares ,week(fc.date) as week_no,
sum(shares)*100 / sum(sum(shares)) over() as shares_perc 
from fact_content fc 
group by week_no 
)
select week_no,total_shares,shares_perc from cte_1 ;
```
![image](https://github.com/user-attachments/assets/abf146f1-8d27-468b-8177-6cbf2ecf666a)

