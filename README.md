# Sparkify Database Warehouse

## Overview
A music streaming startup, Sparkify, has grown their user base and song database and want to move their processes and data onto the cloud. Their data resides in S3, in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

As their data engineer, you are tasked with building an ETL pipeline that extracts their data from S3, stages them in Redshift, and transforms data into a set of dimensional tables for their analytics team to continue finding insights into what songs their users are listening to.

This projects seeks to achieve the above goal by providing the schema and ETL to create and populate a data warehouse
in the cloud for analytics. 
An AWS data pipeline has been built, utilizing Amazon Redshift for staging tables and PostgreSQL for the data warehouse itself. This pipeline feeds data pulled from Amazon S3 into the warehouse. The data warehouse utilizes a star schema structure to facilitate efficient querying by the Sparkify team, allowing them to analyze user activity within their app, including song listening trends.

## Quick start
To access AWS, you need to do in AWS the following:

* create IAM user 
* create IAM role with AmazonS3ReadOnlyAccess access rights
* get ARN
* create and run Redshift cluster 

From the projects templates provided, fill in the required details in dwh.cfg

## Running the scripts
After installing python3 + AWS SDK (boto3) libraries and dependencies, run from command line:

* `python3 create_tables.py` 
* `python3 etl.py`

## The Database
Sparkify analytics database schema has a star design. Star design means that it has one fact table having business data, and supporting dimension tables. 
The fact table answers one of the key questions: what songs users are listening to. DB schema is the following:

![SparkifyDB schema as ER Diagram](./Udacity-DEND-C3-Project3-ERD-20190517v1.png)

_*SparkifyDB schema as ER Diagram.*_

### Fact Table

* **songplays**: song play data together with user, artist, and song info (songplay_id, start_time, user_id, level, song_id, artist_id, session_id, location, user_agent)

### Dimension Tables

* **users**: user info (columns: user_id, first_name, last_name, gender, level)
* **songs**: song info (columns: song_id, title, artist_id, year, duration)
* **artists**: artist info (columns: artist_id, name, location, latitude, longitude)
* **time**: detailed time info about song plays (columns: start_time, hour, day, week, month, year, weekday)

## AWS Redshift set-up

AWS Redshift is used in ETL pipeline as the DB solution. Used set-up in the Project-3 is as follows:

* Cluster: 2x dc2.large nodes
* Location: US-West-2 (as Project-3's AWS S3 bucket)

### Staging tables

* **staging_events**: log data telling what users have done (columns: log_id, artist, auth, firstName, gender, itemInSession, lastName, length, level, location, method, page, registration, sessionId, song, status, ts, userAgent, userId)
* **staging_songs**: song data about songs and artists (columns: num_songs, artist_id, artist_latitude, artist_longitude, artist_location, artist_name, song_id, title, duration, year)


## Running

**Project has two scripts:**

* **create_tables.py**: This script drops existing tables and creates new ones.
* **etl.py**: This script uses data in s3, processes it, and inserts the processed data into the database.

### Run create_tables.py

Type to command line:

`python3 create_tables.py`

* All tables are dropped.
* New tables are created: 2x staging tables + 4x dimensional tables + 1x fact table.
* Output: Script writes _"Tables dropped successfully"_ and _"Tables created successfully"_ if all tables were dropped and created without errors.

### Run etl.py

Type to command line:

`python3 etl.py`

* Script executes AWS Redshift COPY commands to insert source data (JSON files) to DB staging tables.
* From staging tables, data is further inserted to analytics tables.
* Script writes to console the query it's executing at any given time and if the query was successfully executed.
* In the end, script tells if whole ETL-pipeline was successfully executed.

Output: raw data is in staging_tables + selected data in analytics tables.

## Example queries

* Get users and songs they listened at particular time. Limit query to 1000 hits:

```
SELECT  sp.songplay_id,
        u.user_id,
        s.song_id,
        u.last_name,
        sp.start_time,
        a.name,
        s.title
FROM songplays AS sp
        JOIN users   AS u ON (u.user_id = sp.user_id)
        JOIN songs   AS s ON (s.song_id = sp.song_id)
        JOIN artists AS a ON (a.artist_id = sp.artist_id)
        JOIN time    AS t ON (t.start_time = sp.start_time)
ORDER BY (sp.start_time)
LIMIT 1000;
```

* Get count of rows in staging_songs table:

```
SELECT COUNT(*)
FROM staging_songs;
```

* Get count of rows in songplays table:

```
SELECT COUNT(*)
FROM songplays;
```