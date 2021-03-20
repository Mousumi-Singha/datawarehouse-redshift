# Data Warehouse

## Purpose

A music streaming startup, Sparkify, has their data in Amazon S3. The data is contained in two directories:
1. Directory of JSON metadata for the songs in their app, and
2. Directory of JSON logs on user activity in their app.

The goal of this project is to build an ETL pipeline to:
- Extract data from S3,
- Store the data in Redshift staging tables, and
- Transform data into a set of dimensional tables in Redshift.

The final tables will be used by Sparkify's analytics team to gain insight into the songs that their users are listening to.

## Dataset

The first [dataset](s3://udacity-dend/song_data) contains the song data in JSON format. Files are partitioned by first three letters of each song's track ID e.g. `song_data/A/B/C/TRABCEI128F424C983.json`. Example of a single song file:
```
{
    "num_songs": 1, 
    "artist_id": "ARJIE2Y1187B994AB7", 
    "artist_latitude": null, 
    "artist_longitude": null, 
    "artist_location": "", 
    "artist_name": "Line Renaud", 
    "song_id": "SOUPIRU12A6D4FA1E1", 
    "title": "Der Kleine Dompfaff", 
    "duration": 152.92036, 
    "year": 0
}
```

The second [dataset](s3://udacity-dend/log_data) contains the log metadata. The log files are paratitioned by year and month e.g., `log_data/2018/11/2018-11-12-events.json`. Example entry in a log file:
```
{
    "artist": "Pavement",
    "auth": "Logged in",
    "firstName": "Sylvie",
    "gender": "F",
    "iteminSession": 0,
    "lastName": "Cruz",
    "length": 99.16036,
    "level": "free",
    "location": "Kiamath Falls, OR",
    "method": "PUT",
    "page": "NextSong",
    "registration": 1.540266e+12,
    "sessionId": 345,
    "song": "Mercy: The Laundromat",
    "status": 200,
    "ts": 1541990258796,
    "userAgent": "Mozzilla/5.0...",
    "userId": 10
}
```

## Schema for Song Play Analysis

# Staging tables

`staging_events`: event data telling what users have done (columns: event_id, artist, auth, firstName, gender, itemInSession, lastName, length, level, location, method, page, registration, sessionId, song, status, ts, userAgent, userId)

`staging_songs`: song data about songs and artists (columns: num_songs, artist_id, artist_latitude, artist_longitude, artist_location, artist_name, song_id, title, duration, year)

# Final tables
The Redshift tables are created for the analytics team and they will be a star schema which is optimized for queries on song play analysis. The star schema contains the following fact and dimension tables:
1. `songplay` (fact) - records event data for each song play
    + `songplay_id`
    + `start_time`
    + `user_id`
    + `level`
    + `song_id`
    + `artist_id`
    + `session_id`
    + `location`
    + `user_agent`
2. `users` (dimension) - users in the app
    + `user_id`
    + `first_name`
    + `last_name`
    + `gender`
    + `level`
3. `song` (dimension) - songs in music database
    + `song_id`
    + `title`
    + `artist_id`
    + `year`
    + `duration`
4. `artist` (dimension) - artists in music database
    + `artist_id`
    + `name`
    + `location`
    + `lattitude`
    + `longtitude`
5. `time` (dimension) - timestamps of records in `songplays`
    + `start_time`
    + `hour`
    + `day`
    + `week`
    + `month`
    + `year`
    + `weekday`
    
## Prerequisites
-Python 3
-AWS SDK (boto3) (+ dependencies) to enable scripts and Jupyter to connect to AWS Redshift DB.
- jupyter (+ dependencies) to enable Jupyter Notebook.
-`configparser` and `psycopg2` are available.
-A Redshift cluster

## ETL Pipeline

1. Create staging and final tables in Redshift.
2. Copy data from s3 into Redshift staging tables.
3. Insert data from staging into final tables in Redshift.

## How to run

Although the data-sources are provided by two S3 buckets the only thing we need is an AWS Redshift Cluster up and running

And of course Python

Notes:

In this project a Redshift dc2.large cluster with 1 node has been created.

# Redshift Cluster Description

Redshift cluster properties:
- Node type: `dc2.large`
- Number of nodes: 1
- Cluster identifier: `redshift-cluster-1`
- Database name: `dev`
- Database port: 5439
- Master username: `awsuser`
- IAM role: `myRedshiftRole`
    * Allows Redhshift to communicate with S3 with `AmazonS3ReadOnlyAccess` role
- Default VPC: `vpc-4b0c2a33`
- Security group: `redshift_security_group`
    * Opens TCP port for all incoming traffic in order to connect with database

After opening terminal session, set your filesystem on project root folder
and insert these commands in order to run the demo:

This will create our tables, this must run first
`python create_tables.py`

And the following will execute our ETL process
`python etl.py`

## Files

`create_tables.py` - This script will drop old tables (if exist) ad re-create new tables.
`etl.py` - This script executes the queries that extract JSON data from the S3 bucket and ingest them into Redshift.
`sql_queries.py` - This file contains the CREATE, DROP, COPY and INSERT statements for the staging and final Redshift tables.
`dhw.cfg` - Configuration file used that contains info about Redshift, IAM and S3.
`README.md` - This very document which mentions the specifics of the project.


### Final Tables Verification

We verified the final tables contain the correct data using Redshift query editor. 

Get count of rows in `songplay` table:

SELECT COUNT(*) FROM songplay;

245719 rows

Sample entries in `songplay`:
```
songplay_id,start_time,user_id,level,song_id,artist_id,session_id,location,user_agent
6,1543256734796,92,free,SONQBUB12A6D4F8ED0,ARFCUN31187B9AD578,938,Palestine, TX,Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:31.0) Gecko/20100101 Firefox/31.0
22,1542824952796,97,paid,SOSMXVH12A58A7CA6C,AR6PJ8R1187FB5AD70,817,Lansing-East Lansing, MI,"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/37.0.2062.94 Safari/537.36"
```

Get count of unique users from `users` table:

SELECT COUNT(DISTINCT user_id) FROM users;

104 rows

Sample entries in `users`:
```
user_id,first_name,last_name,gender,level
66,Kevin,Arellano,M,free
15,Lily,Koch,F,paid
```
Get count of rows in `song` table:

SELECT COUNT(*) FROM song;

384824 rows

Sample entries in `song`:
```
song_id,title,artist_id,year,duration
SOZCTXZ12AB0182364,Setanta matins,AR5KOSW1187FB35FF4,0,269.58322
SOBYAKJ12AB017C6E2,Broken (LP Version),ARAO91X1187B98CCA4,2002,259.91791
```

Get count of rows in `artist` table:

SELECT COUNT(*) FROM artist;

45266 rows

Sample entries in `artist`:
```
artist_id,name,location,latitude,longitude
ARL26PR1187FB576E5,Camera Obscura,Glasgow, Scotland,,
AR0WQ0N1187FB3AAB9,The Accüsed,,,
```

Get count of rows in `time` table:

6813 rows

Sample entries in `time`:
```
start_time,hour,day,week,month,year,weekday
2018-11-02 01:25:34.000000,1,2,44,11,2018,2
2018-11-02 02:42:48.000000,2,2,44,11,2018,2
```

## Staging queries:

Get count of rows in staging_songs table:

SELECT COUNT(*) FROM staging_songs;

385252 rows

Get count of rows in staging_events table:

SELECT COUNT(*) FROM staging_events;

8056 rows

### Example queries:

Get users and songs they listened at particular time. Limit query to 1000 rows:

SELECT  sp.songplay_id,
        u.user_id,
        s.song_id,
        u.last_name,
        sp.start_time,
        a.name,
        s.title
FROM songplay AS sp
        JOIN users   AS u ON (u.user_id = sp.user_id)
        JOIN song   AS s ON (s.song_id = sp.song_id)
        JOIN artist AS a ON (a.artist_id = sp.artist_id)
        JOIN time    AS t ON (t.start_time = sp.start_time)
ORDER BY (sp.start_time)
LIMIT 1000;