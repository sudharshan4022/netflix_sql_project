# Netflix Movies and TV Shows Data Analysisusing SQL

![Netflix Logo](https://github.com/sudharshan4022/netflix_sql_project/blob/main/netflix.logo.png)

## Objective

- Analyze the distribution of content types(Movies Vs TV Shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteris and keywords.

## Dataset

The data this project is  sourced from the kaggle dataset:

## Dataset link:##(https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
 
## Schema

--netfliex project
drop table if exists netflix;
create table  netflix
(
	show_id	varchar(6),
	type varchar(10),
	title varchar(150),	
	director varchar(215),	
	casts varchar(800),
	country	varchar(150), 
	date_added	varchar(50),
	release_year varchar(50),
	rating varchar(10),
	duration varchar(15),	
	listed_in varchar(100),
	description varchar(250)
);
select * from netflix;

select
 count(*) as total_content
from netflix; 

select
 distinct type
from netflix; 

-- 15 Business Problems & Solutions

###1. Count the number of Movies vs TV Shows

select
    type,
    count (*) as total_content
from netflix
group by type

###2. Find the most common rating for movies and TV shows

select 
    type,
    rating
from
(
	select
	  type,
	  rating,
	  count(*),
	  rank() over(partition by type order by count(*) desc) as ranking
	from netflix
	group by 1,2
) as t1
where
  ranking = 1

###3. List all movies released in a specific year (e.g., 2020)

select  * from netflix
where 
    type ='Movie'
	and
    release_year = '2020'

###4. Find the top 5 countries with the most content on Netflix

select
    --unnest(takes an array as input and transforms it into a table-like structure. Each element of the array becomes a separate row in the resulting output)
    unnest(string_to_array(country,',')) as new_country,
    count(show_id) as total_content
from netflix
group by 1
order by 2 desc
limit 5

###5. Identify the longest movie

select * from netflix
where
   type='Movie'
   and
   duration =(select max(duration) from netflix)

###6. Find content added in the last 5 years

select
    * 
from netflix
where 
    to_date(date_added,'Month DD,YYYY') >= current_date -interval '5 years';

-- select current_date - interval '5 years'(this for know the last 5 years)
	
###7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

select * from netflix
where director like '%Rajiv Chilaka%'

###8. List all TV shows with more than 5 seasons

select 
     *
 --split_part(duration, ' ',1) as sessions
from netflix
where
    type = 'TV Show'
	and
    split_part(duration, ' ', 1):: numeric > 5 

###9. Count the number of content items in each genre

select 
     unnest (string_to_array(listed_in, ',')) as genre,
	 count (show_id) as total_content
from netflix
group by 1

###10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

--total content 333/972
select 
      extract(year from to_date(date_added, 'Month DD,YYYY')) AS year,
	  count(*) as yearly_content,
	  round(
	  count(*)::numeric/(select count(*) from netflix where country = 'India')::numeric* 100,2 )as avg_content_per_year
from netflix
where country = 'India'
group by 1

11. List all movies that are documentaries

select * from netflix
where
   listed_in like '%Documentaries%'
   
12. Find all content without a director

select * from netflix
where
   director is null
   
13. Find how many movies actor 'Salman Khan' appeared in last 11 years!

select *
from netflix
where 
    casts ilike '%Salman Khan%'
    and 
	release_year::integer > extract(year from current_date) - 11; 
 Objective: Count the number of movies featuring 'Salman Khan' in the last 10 years.
	
14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

select 
--show_id,
--casts,
unnest(string_to_array(casts, ',')) as  actors,
count(*) as total_content
from netflix
where country ilike '%india%'
group by 1
order by  2 desc
limit 10
Objective: Identify the top 10 actors with the most appearances in Indian-produced movies.

15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.

with new_table
as
(
select 
    * , 
    case
	when description ilike '%kill%' or
	     description ilike '%violence%' then 'bad_content'
		 else 'good_content'
	end  category	 
from netflix
)
select
    category,
	count(*) as total_content
from new_table
group by 1
Objective: Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.
Findings and Conclusion
Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.
Geographical Insights: The top countries and the average content releases by India highlight regional content distribution.
Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.
This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
