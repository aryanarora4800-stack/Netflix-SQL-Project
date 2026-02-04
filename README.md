# Netflix-SQL-Project
This project involves a comprehensive analysis of Netflix's global content library using SQL. As an economics student, I focused on identifying market trends, production longevity across different regions, and catalog health to understand the strategic positioning of the platform.

# Objectives
Market Segmentation: Analyze how Netflix balances its library across different countries.
Content Lifecycle: Identify the gap between content release and its arrival on the platform.
Production Trends: Track the growth of Movies vs. TV Shows over the last decade.
Data Integrity: Perform an audit of the dataset to identify missing metadata that could affect recommendation algorithms.

# Dataset
The data is sourced from Kaggle and contains over 8,000 rows of titles including information on directors, cast, country, release year, and duration.

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

-- 1. Identify the Top 5 Countries with the Most "Movie" Content
SELECT 
    country, 
    COUNT(*) as total_movies
FROM netflix
WHERE type = 'Movie' AND country IS NOT NULL
GROUP BY country
ORDER BY total_movies DESC
LIMIT 5;

--2. Analyze the Content Growth Trend for the Last 10 Years
SELECT 
    release_year, 
    COUNT(*) as total_content,
    LAG(COUNT(*)) OVER (ORDER BY release_year) as previous_year_content,
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY release_year) as growth
FROM netflix
WHERE release_year >= 2012
GROUP BY release_year
ORDER BY release_year DESC;

--3. Identify Directors Who Have Directed Both Movies and TV Shows
SELECT director
FROM netflix
WHERE director IS NOT NULL
GROUP BY director
HAVING COUNT(DISTINCT type) > 1;

--4. Identify Recent Content from a Specific Region (e.g., India)
SELECT 
    title, 
    type, 
    release_year, 
    rating
FROM netflix
WHERE country = 'India' 
  AND release_year > 2020
ORDER BY release_year DESC;

--5. Find the Top 3 Ratings for Each Country
WITH RatingCounts AS (
    SELECT country, rating, COUNT(*) as count,
           RANK() OVER (PARTITION BY country ORDER BY COUNT(*) DESC) as rnk
    FROM netflix
    WHERE country IS NOT NULL AND rating IS NOT NULL
    GROUP BY country, rating
)
SELECT country, rating, count
FROM RatingCounts
WHERE rnk <= 3;

--6. Top 10 Most Popular Genres on Netflix
SELECT 
    listed_in AS genre, 
    COUNT(*) AS total_count
FROM netflix
GROUP BY genre
ORDER BY total_count DESC
LIMIT 10;

--7. Detect "Late Arrivals": Content Added 10+ Years After Release
SELECT 
    title, 
    release_year, 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year_added,
    (EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) - release_year) as year_gap
FROM netflix
WHERE date_added IS NOT NULL
  AND (EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) - release_year) > 10
ORDER BY year_gap DESC;

--8. Find Titles Containing Specific Keywords Like "History" or "War"
SELECT title, description
FROM netflix
WHERE description LIKE '%history%' OR description LIKE '%war%'
   OR title LIKE '%history%' OR title LIKE '%war%';

--9. Data Quality Check: Identify Countries with the Most Missing Director Data
SELECT 
    SPLIT_PART(country, ',', 1) AS primary_country, 
    COUNT(*) AS missing_director_count
FROM netflix
WHERE director IS NULL 
  AND country IS NOT NULL
GROUP BY primary_country
ORDER BY missing_director_count DESC
LIMIT 10;

--10. Global Content Executive Snapshot
SELECT 
    COUNT(*) AS total_titles,
    COUNT(DISTINCT rating) AS unique_rating_categories,
    SUM(CASE WHEN type = 'Movie' THEN 1 ELSE 0 END) AS movie_count,
    SUM(CASE WHEN type = 'TV Show' THEN 1 ELSE 0 END) AS tv_show_count,
    ROUND(SUM(CASE WHEN type = 'Movie' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS movie_percentage
FROM netflix;
