---
layout: single
title: The Simpsons by Data - Part 1
---

While exploring Kaggle datasets for my next project idea, I found a really unique dataset that caught my eyes. Someone uploaded data about the popular animation "The Simpsons". I grew up with "The Simpsons" so I had to look closely and see what the data looked like. The dataset had four different csv files with all characters, all locations, episode information for all 27 seasons, and all script lines for each characters in the show. You can find out more about the data <a href='https://www.kaggle.com/wcukierski/the-simpsons-by-the-data'>here</a>.

Without thinking what to do for my project, I decided to work with this dataset. For my previous course projects, I worked with public data like transportation, health, housing, etc. I thought working with "The Simpsons" dataset would be a really interesting one and maybe helpful when I work for a consumer facing media/Internet company in the future. Most importantly, I knew I am going to have a lot of fun working with this one. 

As a first step, I threw csv files into my local MySQL. It was easy for me to slice and dice the data in a format I need for further analysis. Additionally, I scraped writer / director information for each episode and merged them into my episode table. Refer to <a href='https://en.wikipedia.org/wiki/List_of_The_Simpsons_episodes'> Simpsons Episode Information </a>. 

### Exploratory Data Analysis

Let's first take a look at IMDB rating and US viewership of 27 seasons. Early seasons have high ratings & high viewer while later ones are in low ratings and low viewer area. 

![alt text](/images/season_imdb_viewer.png "season stat")


In fact, US viewer number has decreased gradually over the last decade. "The Simpsons" was one of the first TV animations that targeted adults and now there are many other shows in similar content such as "The Family Guy" or "South Park". This could be one reason for the decline but it's really the general trend of US viewer for TV shows. This would need an additional analysis after gathering external data and I will leave this part out for now.


![alt text](/images/simpson_viewership.png "viewership")

Now some SQL for EDA..

```sql
SELECT location_name, count(*) AS script_cnt \
FROM script_lines \
WHERE location_name !='' \
GROUP BY 1 \
ORDER BY 2 DESC \
LIMIT 10
```

![alt text](/images/simpsons_location.png "location")

Top 10 most appeared locations in the show.

```sql
SELECT (SUM(CASE WHEN location_name = 'Simpson Home' THEN 1 ELSE 0 END) * 100.0 / COUNT(id)) As Simpson_Home \
FROM script_lines

Simpson_Home
22.15018

```
22.15% of the time the scene takes place in Simpson Home.

```
SELECT gender, COUNT(*) as Total from characters WHERE gender !='na' GROUP BY 1;

gender 	Total
f 	    71
m 	    252

```

Huge gender inequality in the show.
Let's take a look at number of script lines per character

```
SELECT character_name, count(*) AS script_cnt 
FROM script_lines 
WHERE character_name !='' 
GROUP BY 1 
ORDER BY 2 DESC 
LIMIT 20;
```

These are the characters with top 20 number in script lines.
Visualizing the result with Seaborn

![alt text](/images/simpson_scriptlines.png "character scriptlines")


```sql
SELECT (SUM(CASE WHEN character_name = 'Homer Simpson' THEN 1 ELSE 0 END) * 100.0 / COUNT(id)) As Homer_Script 
FROM script_lines 
WHERE character_name!='';

Homer_Script
21.20362
```
Of course, 22.20% of total scrip lines belong to Homer


```sql
SELECT a.character_name, b.season, count(*) AS script_cnt 
FROM script_lines a JOIN episodes b ON a.episode_id = b.id 
WHERE a.character_name ='Homer Simpson' OR a.character_name = 'Bart Simpson' OR a.character_name = 'Lisa Simpson' OR a.character_name = 'Marge Simpson' 
AND speaking_line='true' 
GROUP BY 1,2 
ORDER BY 2 ASC 
```

![alt text](/images/simpson_scriptlines.png "scriptlines")

Homer outnumbers any of his family members in number of script lines. 

Let's take a look at writers/directors information.
Writers with highest IMDB rating,

```
SELECT a.writer, AVG(b.us_viewers_in_millions) AS average_us_viewership, AVG(b.imdb_rating) AS average_imdb_rating, COUNT(*) AS num_episodes 
FROM writers a JOIN episodes b ON a.production_code = b.production_code \
GROUP BY 1 
HAVING num_episodes > 2 
ORDER BY 3 DESC 
LIMIT 10
```
![alt text](/images/simpson_writer.png "writers")

![alt text](/images/writer_viz.png "writer_viz")

Directors with highest IMDB rating

```
SELECT a.director, AVG(b.us_viewers_in_millions) AS average_us_viewership, AVG(b.imdb_rating) AS average_imdb_rating, COUNT(*) AS num_episodes 
FROM directors a JOIN episodes b ON a.production_code = b.production_code \
GROUP BY 1 
HAVING num_episodes > 2 
ORDER BY 3 DESC 
LIMIT 10
```

![alt text](/images/simpson_directors.png "directors")

![alt text](/images/simpson_director_viz.png "directors_viz")

```sql
SELECT COUNT(DISTINCT writer) AS Total_Writers FROM writers

Total_Writers
127

SELECT COUNT(distinct director) as Num_Directors from directors
Num_Directors
43
```
There have been total 127 writers and 43 directors throughout 27 seasons


### Next Part

I will talk about script line NLP in my next post.
