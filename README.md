# SQL-PROJECT---MOVIES-
This project analyzes a movie dataset using SQL queries. It demonstrates key SQL concepts such as SELECT, JOIN, GROUP BY, ORDER BY, and aggregate functions to retrieve and analyze movie information including genres, ratings, and release data to gain meaningful insights from the database.

-- PROJECT OF MOVIES AND THE ANALYSIS THROUGH SQL:
create database proj;
use proj;
drop database proj;
-- Create 'titles' table
DROP TABLE IF EXISTS titles;
CREATE TABLE titles (
    id VARCHAR(30),
    title VARCHAR(255),
    type VARCHAR(50),
    description TEXT,
    release_year VARCHAR(10),
    age_certification VARCHAR(10),
    runtime VARCHAR(10),
    genres VARCHAR(255),
    production_countries VARCHAR(100),
    seasons VARCHAR(10),
    imdb_id VARCHAR(20),
    imdb_score VARCHAR(10),
    imdb_votes VARCHAR(20),
    tmdb_popularity VARCHAR(20),
    tmdb_score VARCHAR(10)
);

SET GLOBAL local_infile = 1;

LOAD DATA INFILE "C:/titles.csv"
INTO TABLE titles
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;


DROP TABLE IF EXISTS credits;
CREATE TABLE credits (
    person_id VARCHAR(40),
    id VARCHAR(50),
    name VARCHAR(255),
    character_name VARCHAR(500),
    role VARCHAR(100)
);

LOAD DATA INFILE "C:/credits.csv"
INTO TABLE credits
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;


-- QUESTIONS :
-- What were the top 10 movies according to IMDB score?
SELECT title, imdb_score
FROM titles
WHERE type = 'MOVIE'
ORDER BY imdb_score DESC
LIMIT 10;

-- What were the top 10 shows according to IMDB score?
SELECT title, imdb_score
FROM titles
WHERE type = 'SHOW'
ORDER BY imdb_score DESC
LIMIT 10;

-- What were the bottom 10 movies according to IMDB score?
SELECT title, imdb_score
FROM titles
WHERE type = 'MOVIE' AND imdb_score IS NOT NULL
ORDER BY imdb_score ASC
LIMIT 10;

-- What were the bottom 10 shows according to IMDB score?
SELECT title, imdb_score
FROM titles
WHERE type = 'SHOW' AND imdb_score IS NOT NULL
ORDER BY imdb_score ASC
LIMIT 10;

-- What were the average IMDB and TMDB scores for shows and movies?
SELECT type,
       AVG(imdb_score) AS avg_imdb,
       AVG(tmdb_score) AS avg_tmdb
FROM titles
GROUP BY type;

-- Count of movies and shows in each decade
SELECT 
  (release_year DIV 10) * 10 AS decade,
  type,
  COUNT(*) AS total_titles
FROM titles
WHERE release_year IS NOT NULL
GROUP BY decade, type
ORDER BY decade;

-- What were the average IMDB and TMDB scores for each production country?
SELECT production_countries,
AVG(imdb_score) AS avg_imdb,
AVG(tmdb_score) AS avg_tmdb
FROM titles
GROUP BY production_countries
ORDER BY avg_imdb DESC;

-- What were the average IMDB and TMDB scores for each age certification for shows and movies?
SELECT type,
       age_certification,
       AVG(imdb_score) AS avg_imdb,
       AVG(tmdb_score) AS avg_tmdb
FROM titles
GROUP BY type, age_certification
ORDER BY type, avg_imdb DESC;

-- What were the 5 most common age certifications for movies?
SELECT age_certification, COUNT(*) AS count
FROM titles
WHERE type = 'MOVIE'
GROUP BY age_certification
ORDER BY count DESC
LIMIT 5;

-- Who were the top 20 actors that appeared the most in movies/shows
SELECT name, COUNT(*) AS appearances
FROM credits
WHERE role = 'ACTOR'
GROUP BY name
ORDER BY appearances DESC
LIMIT 20;

-- Who were the top 20 directors that directed the most movies/shows?
SELECT name, COUNT(*) AS total_directed
FROM credits
WHERE role = 'DIRECTOR'
GROUP BY name
ORDER BY total_directed DESC
LIMIT 20;

-- Calculating the average runtime of movies and TV shows separately
SELECT type, AVG(runtime) AS avg_runtime
FROM titles
GROUP BY type;

-- Finding the titles and directors of movies released on or after 2010
SELECT t.title, c.name AS director
FROM titles t
JOIN credits c 
ON t.id = c.id
WHERE t.type = 'MOVIE' AND c.role = 'DIRECTOR' AND t.release_year >= 2010;

-- Which shows on Netflix have the most seasons?
SELECT title, seasons
FROM titles
WHERE type = 'SHOW'
ORDER BY seasons DESC
LIMIT 10;

-- Which genres had the most movies
SELECT genres, COUNT(*) AS count
FROM titles
WHERE type = 'MOVIE'
GROUP BY genres
ORDER BY count DESC
LIMIT 10;

-- Which genres had the most shows?
SELECT genres, COUNT(*) AS count
FROM titles
WHERE type = 'SHOW'
GROUP BY genres
ORDER BY count DESC
LIMIT 10;

-- Titles and Directors of movies with high IMDB scores (>7.5) and high TMDB popularity scores (>80)
SELECT title, imdb_score, tmdb_popularity
FROM titles
WHERE type = 'MOVIE'
  AND imdb_score > 7.5
  AND tmdb_popularity > 80;

-- What were the total number of titles for each year?
SELECT release_year, COUNT(*) AS total_titles
FROM titles
GROUP BY release_year
ORDER BY release_year;

-- Actors who have starred in the most highly rated movies or shows
SELECT c.name, COUNT(*) AS appearances
FROM credits c
JOIN titles t 
ON c.id = t.id
WHERE t.imdb_score >= 8
GROUP BY c.name
ORDER BY appearances DESC
LIMIT 10;

-- Which actors/actresses played the same character in multiple movies or TV shows?
SELECT name, character_name, COUNT(*) AS times_played
FROM credits
WHERE character_name IS NOT NULL
GROUP BY name, character_name
HAVING COUNT(*) > 1
ORDER BY times_played DESC
limit 15;

-- What were the top 3 most common genres?
SELECT genres, COUNT(*) AS total
FROM titles
GROUP BY genres
ORDER BY total DESC
LIMIT 3;

-- Average IMDB score for leading actors/actresses in movies or shows
SELECT name, AVG(imdb_score) AS avg_imdb
FROM credits
JOIN titles USING (id)
WHERE role = 'ACTOR'
GROUP BY name
ORDER BY avg_imdb DESC
LIMIT 20;

SELECT 
    c.name,
    ROUND(AVG(t.imdb_score), 2) AS avg_imdb
FROM credits c
JOIN titles t 
    ON c.id = t.id
WHERE c.role = 'ACTOR'
  AND t.imdb_score IS NOT NULL
GROUP BY c.name
ORDER BY avg_imdb DESC
LIMIT 20;
