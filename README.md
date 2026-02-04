# Netflix Content Strategy & Regional Analytics (SQL)

## Project Overview

This project performs a deep-dive analysis of Netflix's global content library using SQL. As an economist, I approached this dataset with the goal of uncovering the strategic logic behind content distribution, production longevity, and regional market focus. By transforming raw data into actionable business intelligence, this analysis highlights how Netflix balances its library to cater to diverse global audiences.

## Objectives

* **Market Analysis:** Categorize content distribution across key global regions.
* **Production Trends:** Track the growth of the catalog over the last decade.
* **Content Lifecycle:** Measure the gap between a title's original release and its arrival on the platform.
* **Data Quality Audit:** Identify metadata gaps that could impact search algorithms and discoverability.
* **Executive Snapshot:** Provide high-level KPIs regarding content mix and audience targeting.

## Database Schema

```sql
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

```

## Business Problems and Solutions

### 1. Top 5 Countries with the Most "Movie" Content

**Business Insight:** Identifies the primary film-producing hubs for Netflix, helping to understand regional licensing priorities.

```sql
SELECT 
    country, 
    COUNT(*) as total_movies
FROM netflix
WHERE type = 'Movie' AND country IS NOT NULL
GROUP BY country
ORDER BY total_movies DESC
LIMIT 5;

```

### 2. Content Growth Trend (Last 10 Years)

**Business Insight:** Tracks the velocity of content acquisition to visualize how the platform has scaled since 2012.

```sql
SELECT 
    release_year, 
    COUNT(*) as total_content,
    LAG(COUNT(*)) OVER (ORDER BY release_year) as previous_year_content,
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY release_year) as growth
FROM netflix
WHERE release_year >= 2012
GROUP BY release_year
ORDER BY release_year DESC;

```

### 3. Directors with Versatile Content (Movies & TV Shows)

**Business Insight:** Highlights versatile creators who contribute across both formats, often indicating high-value talent relationships.

```sql
SELECT director
FROM netflix
WHERE director IS NOT NULL
GROUP BY director
HAVING COUNT(DISTINCT type) > 1;

```

### 4. Regional Focus: Emerging Market (India)

**Business Insight:** Analyzes the content pipeline in high-growth regions by filtering for new titles (post-2020).

```sql
SELECT 
    title, 
    type, 
    release_year, 
    rating
FROM netflix
WHERE country = 'India' 
  AND release_year > 2020
ORDER BY release_year DESC;

```

### 5. Top 3 Ratings for Each Country

**Business Insight:** Uses window functions to identify the dominant target audience (Kids, Teens, Adults) in each market.

```sql
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

```

### 6. Top 10 Most Popular Genres

**Business Insight:** Provides a high-level view of the most frequent genres, indicating where content competition is highest.

```sql
SELECT 
    listed_in AS genre, 
    COUNT(*) AS total_count
FROM netflix
GROUP BY genre
ORDER BY total_count DESC
LIMIT 10;

```

### 7. "Late Arrivals" (10+ Year Gap)

**Business Insight:** Identifies licensed "legacy" content versus new "originals" by calculating the gap between release year and add date.

```sql
SELECT 
    title, 
    release_year, 
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year_added,
    (EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) - release_year) as year_gap
FROM netflix
WHERE date_added IS NOT NULL
  AND (EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) - release_year) > 10
ORDER BY year_gap DESC;

```

### 8. Thematic Analysis: History and War

**Business Insight:** Uses keyword search to quantify the volume of content within specific historical or thematic niches.

```sql
SELECT title, description
FROM netflix
WHERE description LIKE '%history%' OR description LIKE '%war%'
   OR title LIKE '%history%' OR title LIKE '%war%';

```

### 9. Data Integrity Audit: Missing Director Metadata

**Business Insight:** A data quality check to identify which regions have the most missing "Director" information, impacting search discoverability.

```sql
SELECT 
    SPLIT_PART(country, ',', 1) AS primary_country, 
    COUNT(*) AS missing_director_count
FROM netflix
WHERE director IS NULL 
  AND country IS NOT NULL
GROUP BY primary_country
ORDER BY missing_director_count DESC
LIMIT 10;

```

### 10. Executive Dashboard (Master Query)

**Business Insight:** The ultimate summary query providing a "Content Mix Ratio" (Movies vs. TV Shows) to assess library health.

```sql
SELECT 
    COUNT(*) AS total_titles,
    COUNT(DISTINCT rating) AS unique_ratings,
    SUM(CASE WHEN type = 'Movie' THEN 1 ELSE 0 END) AS movie_count,
    SUM(CASE WHEN type = 'TV Show' THEN 1 ELSE 0 END) AS tv_show_count,
    ROUND(SUM(CASE WHEN type = 'Movie' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS movie_percentage
FROM netflix;

```

## Key Findings

* **Library Strategy:** The catalog is heavily focused on Movies, though TV Shows are a significant driver of long-term engagement.
* **Regional Variation:** Emerging markets show a faster turnover of new content, while established markets maintain a larger volume of legacy titles.
* **Metadata Gaps:** Identifying missing director info revealed regional variances in data quality, which is vital for optimizing recommendation engines.

## Credits

* **Mentor:** Guidance and project inspiration provided by @najirh
* **Dataset:** Netflix Titles dataset sourced via Kaggle.

---

**Author:** Aryan Arora

*M.A. Economics Student, IIFT New Delhi (2025-27)* *Focusing on the intersection of Economic Theory and Data Analytics.*
