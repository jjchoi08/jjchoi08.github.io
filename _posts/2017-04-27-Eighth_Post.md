---
layout: single
title: Collaborative Filtering with SQL
---

I interviewed with few tech companies last year for data analyst / data scientist roles. I came across some interesting SQL questions as their data challenges. Although I've used SQL for many years for reports/dashboard management, some of them were very challenging. I could see SQL still being widely used in data science field. I wanted to post about one interesting SQL problem that I received as a data challenge.


### Simple Recommendation System

The company wanted me write a SQL query for a recommendation system. In a real world, companies would use more complex and convienient tools for creating recommendation system but this could be a good SQL challenge to test candidates' skill. 



Tables

```
playlist
--------
user_id
playlist_id

listings
--------
playlist_id
track_id
position
track_duration

plays
--------
timestamp
user_id
track_id
playlist_id
listening_duration

```

playlists table with columns:
- user_id: the user who uploaded the playlist
- playlist_id: the unique identifier of the playlist

listings table with columns:
- playlist_id: the unique identifier of the playlist
- track_id: the unique identifier of the track
- position: the position of this track in the playlist
- track_duration: the length of the track recording (in milliseconds)

plays table with columns:
- timestamp: the time the play occurred
- user_id: the user who played the given track
- track_id: the unique identifier of the track
- playlist_id: the unique identifier of the playlist (set to 0 if the play did not happen in a playlist)
- listening_duration: the duration that the user listened to the track (in milliseconds)

### Challlege: Recommend five playlist that a user haven't heard before

My approach would be using collaborative filtering and find playlists that contains music that the user has listened to the most. Based on this, I can come up with some kind of rank.I will create a rank_table with users who might have similar taste in music.

Let's assume we are recommending 5 playlists for user_id = 1 (let's call it user x).
 

Rank is the total number of songs that a user listened to, in same playlists with the user x's.
rank_table would look something like this,

```
SELECT * FROM rank_table

user_id  |  Rank
----------------
2        |    10
3        |    6
4        |    6

```

```
 1) The first table

reco.timestamp | reco.userid | reco.trackid | reco.playlistid | reco.duration | rank_table.rank
3/2            | 2           | 2            | 3               | 100           | 10
3/2            | 3           | 3            | 5               | 100           | 6
3/2            | 4           | 4            | 7               | 100           | 6
```

```
2) The Second table

reco.timestamp | reco.userid | reco.trackid | reco.playlistid | reco.duration | rank_table.rank | userx.reco.timestamp | userx.userid | userx.trackid | userx.playlistid | userx.duration 
3/2            | 2           | 2            | 3               | 100           | 10              | 3/2                  | 1            | 2             | 3                | 100            
3/2            | 3           | 3            | 5               | 100           | 6               | 3/3                  | 1            | 2             | NULL             | 100 
3/2            | 4           | 4            | 7               | 100           | 6               | 3/5                  | 1            | 3             | 7                | 100 
3/2            | 3           | 7            | 9               | 100           | 7               | 3/6                  | 1            | 9             | NULL             | 100
```

When group by reco.playlist_id and add rank, we have playlists that people listened to the most in decending order.
Since we only selected those with userx.playlistid = NULL, user x (userid=1) has never listened to those playlists.



My aswer would be..

```sql
WITH rank_table AS
(
	SELECT b.user_id, count(*) AS Rank
	FROM plays a
	JOIN plays b ON a.playlist_id = b.playlist_id 
	AND a.user_id != b.user_id
	WHERE a.user_id = 1
	GROUP BY b.user_id
	ORDER BY count(*) DESC
)

SELECT reco.playlist_id, sum(rank_table.rank) AS total_rank
FROM rank_table
JOIN plays reco ON rank_table.user_id = plays.user_id
LEFT JOIN plays userx ON userx.user_id = 1 AND userx.playlist_id = reco.playlist_id
WHERE userx.playlist_id IS NULL
GROUP BY reco.playlist_id
ORDER BY total_rank DESC
LIMIT 5;
```

It's a challenging but interesting problem. I've never thought of using collaborative filtering with SQL. I am making transition to data science from SQL focused business analyst so this kind of problem helps me think more analytically.

Feel free to send me any SQL challenges similar to this one!

