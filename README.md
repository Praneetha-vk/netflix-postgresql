# netflix-postgresql

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset.

--netflix project

DROP TABLE IF EXISTS netflix;
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

SELECT * FROM netflix;

--Q1. Count the Number of Movies vs TV Shows

SELECT 
    type,
    COUNT(*) as Total_shows
FROM netflix
GROUP BY type;

--Q2. Find the Most Common Rating for Movies and TV Shows


SELECT 
	type,
	rating
FROM (
	SELECT 
		type,
		rating,
		COUNT(*),
		RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS Ranking 
	FROM netflix
	GROUP BY 1,2
) AS T1
WHERE 
		Ranking=1


--OR
WITH RatingCount AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankingRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCount
)
SELECT 
    type,
    rating AS most_common_rating
FROM RankingRatings
WHERE rank = 1;

--Q3. Find the least rating given

SELECT 
	type,
	rating
FROM
(
SELECT 
		type,
		rating,
		COUNT(*),
		ROW_NUMBER() OVER (PARTITION BY type ORDER BY COUNT(*)) AS Ranking 
	FROM netflix
	GROUP BY 1,2
)as T2
WHERE Ranking=1;

--Q4 List all movies released in a specific year (EX: 2021)

SELECT * 
FROM netflix
WHERE 
	type='Movie'
	AND 
	release_year=2021

--Q5 Find the top 5 countries with the most content(mvs/shows) on Netflix

SELECT 
	UNNEST(STRING_TO_ARRAY(country,',')) as new_Country,
	COUNT(show_id) as overall_content
FROM netflix
GROUP BY 1
ORDER BY overall_content DESC 
LIMIT 5

--Q6  Identify the longest movie

SELECT *
FROM netflix
WHERE type = 'Movie' AND duration is NOT NULL 
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;

--Q7 Find all the movies/TV shows by a particular director

SELECT * FROM
(
SELECT 
		*,
		UNNEST(STRING_TO_ARRAY(director,',')) as director_name
	FROM netflix
)
WHERE director_name='Toshiya Shinohara'

--Q8 List all TV shows with more than 5 seasons

SELECT * 
FROM netflix
WHERE 
	type='TV Show'
	AND
	SPLIT_PART(duration,' ',1):: INT > 5 

--Q9 Count the number of content items in each genre

SELECT 
	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as genre,
	COUNT(*) as total_content
FROM netflix
GROUP BY 1

--Q10 List all movies that are documentaries

SELECT * FROM netflix
WHERE listed_in LIKE '%Documentaries'


--Q11 Find all content without a director
SELECT * FROM netflix
WHERE director IS NULL


--Q12 Find how many movies actor 'Kamal Hassan' appeared in last 10 years!

SELECT * FROM netflix
WHERE 
	casts LIKE '%Kamal Hassan%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10





This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

	
