# Netflix Movies and TV Shows Data Analysis using SQL
<img src="https://github.com/user-attachments/assets/409775bd-15f5-4a14-bf79-f9b948291851">


# Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

# Dataset
<p>The data for this project is sourced from the Kaggle dataset:</p>
Dataset Link: https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download

# Schema
```sql
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

# Business Problems and Solutions
**1. Count the number of Movies vs TV Shows**
```sql
select type,count(type) from netflix1 group by type
```

**2. Find the most common rating for movies and TV shows**
```sql
with cte as(
select type,rating,count(rating),rank() over(partition by type order by count(*) desc) as rank_num
from netflix1 group by 1,rating order by 3 desc)
select type,rating  from cte where rank_num=1
```
**3. List all movies released in a specific year (e.g., 2020)**
```sql
select * from netflix1 where  release_year=2020 and type='Movie'
```
**4. Find the top 5 countries with the most content on Netflix**
```sql
select 
unnest(string_to_array(country,',')) as new_country,
count(show_id) from netflix1 group by 1 order by 2 desc limit 5
```

**5. Identify the longest movie**
```sql
select title from netflix1 where type='Movie' and duration =(select max(duration) from netflix1)
```
**6. Find content added in the last 5 years**
```sql
select * from netflix1 where to_date(date_added, 'month ,year') >= CURRENT_DATE-INTERVAL '5 YEARS'
```

**7. Find all the movies/TV shows by director 'Rajiv Chilaka'!**
```sql
select * from netflix1 where  director like '%Rajiv Chilaka%'
```

**8. List all TV shows with more than 5 seasons**
```sql
select * from netflix1 where type='TV Show' and split_part(duration, ' ',1)::numeric > 5
```

**9. Count the number of content items in each genre**
```sql
select count(listed_in), unnest(string_to_array(listed_in,',')) as new_genere from netflix1 
group by 2
```

**10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!**
```sql
select count(*), extract(year from to_date(date_added,'Month dd,yyyy')) as years,
round(count(*)::numeric/(select count(*) from netflix1 where country='India')::numeric *100,2)
 from netflix1 where country='India' group by 2 order by 3 desc limit 5
```

**11. List all movies that are documentaries**
```sql
select * from netflix1 where listed_in ilike '%documentaries%'
```

**12. Find all content without a director**
```sql
select * from netflix1 where director is null
```

**13. Find how many movies actor 'Salman Khan' appeared in last 10 years!**
```sql
select * from netflix1 where casts1 like '%Salman Khan%' and release_year > extract(year from current_date)-10
```

**14. Find the top 10 actors who have appeared in the highest number of movies produced in India.**
```sql
select count(show_id),unnest(string_to_array(casts1,',')) from netflix1 where 
country='India' an group by 2 order by 1 desc limit 10
```

**15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.**
```sql
with cte as(
select *,
      case
	  when description ilike '%kill%' or description ilike '%violence%' then 'Bad'
	  else 'Good' 
	  end as content_type
	  from
netflix1)
select content_type,count(*) from cte  group by 1
```
