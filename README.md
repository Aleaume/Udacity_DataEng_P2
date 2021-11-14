# Udacity_DataEng_P2
Udacity Course, Data Engineering Nanodegree, 2nd Project, Data Modeling with Apache Cassandra

## Requirements

- install apache cassandra ('cassandra' package)
- have the jupyter notebook in the same folder as the "event_data" folder containing the input csv files.

## Overview

In this project the goal is to design & create the appropriate database with its tables for a music streaming app (sparkify). 
Once done the task is to set up an ETL pipeline to ingest and store in this newly DB data made available in CSV, containing info on logs of listening activities from users with the song & artist listned to. 

The database is a NoSQL (Non-relation) DB with the DBMS Apachae Cassandra.

### Architecture

The project makes use of a central jupyter notebook to accomplish most of the tasks, as seen in this diagramm:

![image](https://user-images.githubusercontent.com/32632731/141684769-bc3561d3-facf-4929-a787-de7550de5b7d.png)


### Dataset

There is only one dataset in this project that is stored in a folder named "event_data".
The folder is made of csv files containing the logs of music listned by users of the music app.

Their naming goes as follow: yyyy-mm-dd-events.csv

In order to work more easily with those multiple files, the first part of the notebook takes care of merging all those files in a single file at the root of the working directory: "event_datafile_new.csv".

First a list of all the files' filepath is created and then further worked:

```python

# initiating an empty list of rows that will be generated from each file
full_data_rows_list = [] 
    
# for every filepath in the file path list 
for f in file_path_list:

# reading csv file 
    with open(f, 'r', encoding = 'utf8', newline='') as csvfile: 
        # creating a csv reader object 
        csvreader = csv.reader(csvfile) 
        next(csvreader)
        
 # extracting each data row one by one and append it        
        for line in csvreader:
            #print(line)
            full_data_rows_list.append(line) 
            
# uncomment the code below if you would like to get total number of rows 
print(len(full_data_rows_list))
# uncomment the code below if you would like to check to see what the list of event data rows will look like
#print(full_data_rows_list)

# creating a smaller event data csv file called event_datafile_full csv that will be used to insert data into the \
# Apache Cassandra tables
csv.register_dialect('myDialect', quoting=csv.QUOTE_ALL, skipinitialspace=True)

with open('event_datafile_new.csv', 'w', encoding = 'utf8', newline='') as f:
    writer = csv.writer(f, dialect='myDialect')
    writer.writerow(['artist','firstName','gender','itemInSession','lastName','length',\
                'level','location','sessionId','song','userId'])
    for row in full_data_rows_list:
        if (row[0] == ''):
            continue
        writer.writerow((row[0], row[2], row[3], row[4], row[5], row[6], row[7], row[8], row[12], row[13], row[16]))


```

Note: the part I of the notebook is readily provided by the Udayity Lab and no need for edit is required by the sutdents.

The result is a filtered & ready to use csv file with a similar format:

![image](https://user-images.githubusercontent.com/32632731/141685174-18985439-8e0f-4542-b15e-fb10d7d5b4ce.png)


## Database Creation & Queries

In order to design the Databse that will support this application we need to understand how will it be used / queried.

3 Queries have been provided for that, they go as follow:

1. Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4
2. Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182
3. Give me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

Based on the principle 1 query 1 table, we will proceed to a high denormalization of data and create 3 tables (see tables).

### Database

First we create a Database, or more precisely a Keyspace and make sure to create only if no other DB with the similar naming exists:

```python

try:
    session.execute("""
    CREATE KEYSPACE IF NOT EXISTS sparkify 
    WITH REPLICATION = 
    { 'class' : 'SimpleStrategy', 'replication_factor' : 1 }"""
)

except Exception as e:
    print(e)

```

and then set the Keyspace to be used in the next steps:

```python
try:
    session.set_keyspace('sparkify')
except Exception as e:
    print(e)

```

### Table Creation

As discussed earlier we will go ahead with creating 3 tables with a single query focus for each:

![image](https://user-images.githubusercontent.com/32632731/141685548-3e3f599f-bf5b-435f-8c46-801b833b9e01.png)

#### sessions

Perfect to answer quickly to the query 1, we have here a composite key made of the sessionid and the iteminsession to make sure:

- this a unique set
- the sessionid and iteminsession can be part of the WHERE clause

```python

query = "CREATE TABLE IF NOT EXISTS sessions "
query = query + "(sessionid int, iteminsession int, artist text, duration float, song text, PRIMARY KEY (sessionid, iteminsession))"

session.execute(query)

```

#### users

For the 2nd query we create a table with a focus on users where user id and session id along with iteminsession make a composite key for:

- unique record set
- easy WHERE clause on userid and sessionid
- oder by iteminsession

```python

query = "CREATE TABLE IF NOT EXISTS users "
query = query + "(userid int, sessionid int, iteminsession int, artist text, song text, firstname text, lastname text, PRIMARY KEY (userid, sessionid, iteminsession))"

session.execute(query)

```

#### songs

For the last query we create a table with a focus on songs where song of course but also userid and sessionid consitute a composite key, this gives:

- an effective way to use WHERE clause song
- and also making sure the record is unique with the other 2 parameters

```python

query = "CREATE TABLE IF NOT EXISTS songs "
query = query + "(song text, userid int, sessionid int, firstname text, lastname text, PRIMARY KEY (song, userid, sessionid))"

session.execute(query)
                    

```

### Queries



## ETL (Extract Transform Load)

## Improvement suggestions / Additional work
