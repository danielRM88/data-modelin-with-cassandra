# Data Modeling with Apache Cassandra

## Introduction

Example project to practice concepts about data modeling with Apache Cassandra.

## Problem definition

A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analysis team is particularly interested in understanding what songs users are listening to. Currently, there is no easy way to query the data to generate the results, since the data reside in a directory of CSV files on user activity on the app.

## Solution

An ETL pipeline is created to process the files and merge them all together into one single one.
The resulting file will be used to populate the Apache Cassandra tables. The tables will later be used to perform the required queries by the analytics team.

The following shows a preview of the resulting csv file:

![Result](images/image_event_datafile_new.jpg)

### Modeling the database

Apache Cassandra is a NoSQL database, when doing data modeling with Cassandra, it is important to consider the following:
- JOINS are not supported
- Think queries first
- Optimized for fast writes
- A great strategy is to create one table per query

Cassandra falls under the category of wide column NoSQL database, which means that the queries are limited to the columns that form part of the primary key of the table.

The primary key is made up of just the partition key or with the addition of clustering columns. The partition key will determine the distribution of data across the system.

Important points when designing a primary key:
- Must be unique
- Hashing results in placement on a particular node in the system
- Can be simple (only one column) or composite

The clustering columns will sort the data in ascending order.

The first step when modeling for a Cassandra database is to understand the queries that will be executed. For this project, the analytics team require three main queries:

1.- Select the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4

The query needs 5 columns, artist, song title, song's length, sessionId and itemInSessionId. The WHERE clause requires the sessionId and the itemInSession, which indicates that these two columns should conform the primary key. The order in which they appeared in the WHERE clause is also important. Everything mentioned before leads to the following SQL table definition:
```sql
CREATE TABLE IF NOT EXISTS session_events (
    session_id int,
    item_in_session int,
    artist text,
    song_title text,
    duration float,
    PRIMARY KEY(session_id, item_in_session)
)
```

2.- Select only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182

The query needs 7 columns, artist, song title, first name, last name, user id, sessionId and itemInSessionId. The WHERE clause requires the user id and the sessionId, which indicates that these two columns should conform the primary key, but also the partition key, to make sure that session information for a single user gets stored in the same node within the cluster. It is also important to note that the results should be sorted by itemInSession, i.e. the column should be included as a clustering column in the primary key. Everything mentioned before leads to the following SQL table definition:
```sql
CREATE TABLE IF NOT EXISTS user_session_events (
    user_id int,
    session_id int,
    item_in_session int,
    artist text,
    song_title text,
    first_name text,
    last_name text,
    PRIMARY KEY((user_id, session_id), item_in_session)
)
```

3.- Select every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

The query needs 3 columns, song title, first name and last name. The WHERE clause requires the song title, so it should be included in the primary key, but since there might be multiple users that listen to that song, the user id is also required in the primary key to avoid duplicates. This leads to the following SQL table definition:
```sql
CREATE TABLE IF NOT EXISTS user_events (
    song_title text,
    user_id int,
    first_name text,
    last_name text,
    PRIMARY KEY(song_title, user_id)
)
```

### How to run the project

The ETL pipeline and the data modeling of the database is performed in a python notebook. 

Each step of the notebook is clearly specified and self explanatory, although the explanations of this README should provide the extra context needed to understand the modeling decisions that were chosen for the project.