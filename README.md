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

![image](https://user-images.githubusercontent.com/32632731/141739966-c5e15c06-23a1-4634-844b-9b66a8dc8393.png)


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

For the 2nd query we create a table with a focus on users where user id and session id along make a composite key along with iteminsession as clustering key for:

- unique record set
- easy WHERE clause on userid and sessionid
- oder by iteminsession

```python

query = "CREATE TABLE IF NOT EXISTS users "
query = query + "(userid int, sessionid int, iteminsession int, artist text, song text, firstname text, lastname text, PRIMARY KEY ((userid, sessionid), iteminsession))"

session.execute(query)

```

#### songs

For the last query we create a table with a focus on songs where song of course but also userid consitute a composite key, this gives:

- an effective way to use WHERE clause song
- and also making sure the record is unique with the other parameter userid

```python

query = "CREATE TABLE IF NOT EXISTS songs "
query = query + "(song text, userid int, sessionid int, firstname text, lastname text, PRIMARY KEY (song, userid))"

session.execute(query)
                    

```

### Queries

As mentioned earlier we have 3 queries each using a different table made purposedly:


#### Query 1: Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4


```python

#commented line give the quality check result with checking parameters

#query_case_1 = "SELECT * FROM sessions "
query_case_1 = "SELECT artist, song, duration FROM sessions "
query_case_1 +="WHERE sessionid = 338 AND iteminsession = 4"


try:
    rows = session.execute(query_case_1)
except Exception as e:
    print(e)
    
for row in rows:
    #print (row.artist, row.song, row.duration, row.sessionid, row.iteminsession)
    print (row.artist, row.song, row.duration)

```

#### Query 2:  Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182

```python

#commented line give the quality check result with checking parameters

#query_case_2 = "SELECT * FROM users "
query_case_2 = "SELECT artist, song, firstname, lastname FROM users "
query_case_2 +="WHERE userid = 10 AND sessionid = 182 "


try:
    rows = session.execute(query_case_2)
except Exception as e:
    print(e)
    
for row in rows:
    #print (row.artist, row.song, row.firstname, row.lastname, row.userid, row.sessionid, row.iteminsession)
    print (row.artist, row.song, row.firstname, row.lastname)
    
```

#### Query 3: Give me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

```python

#commented line give the quality check result with checking parameters

#query_case_3 = "SELECT * FROM songs "
query_case_3 = "SELECT firstname, lastname FROM songs "
query_case_3 +="WHERE song = 'All Hands Against His Own'"


try:
    rows = session.execute(query_case_3)
except Exception as e:
    print(e)
    
for row in rows:
    #print (row.song, row.firstname, row.lastname,)
    print (row.firstname, row.lastname)

```

## ETL (Extract Transform Load)

For the ETL part, as mentioned a good part of the work with the files as been already done in the Part I of the notebook. Now we just need to read each rows of the single file and insert properly the values within the right placeholder of the table and this individually for the 3 created tables.

Note: 1 inportant aspect is to convert each values coming as string into the expected format

As an example the ingestion code looks for most of the tables as follow:

```python

file = 'event_datafile_new.csv'

with open(file, encoding = 'utf8') as f:
    csvreader = csv.reader(f)
    next(csvreader) # skip header
    for line in csvreader:
# Assign the INSERT statements into the `query` variable
        query = "INSERT INTO songs (song, userid, sessionid, firstname, lastname)"
        query = query + "VALUES (%s, %s, %s, %s, %s)"
        
        #Assign which column element should be assigned for each column in the INSERT statement.
    

        session.execute(query, (line[9], int(line[10]), int(line[8]), line[1], line[4]))
        #print(type(line[8]), type(line[3]), type(line[0]), type(line[5]), type(line[9]))
      

```


## Improvement suggestions / Additional work

As seen in the code extracts for the SELECT queries I added some quality control checks where not only the requested fields are retrieved and displayed but all the fields part of the composite key. Simply uncomment the lines to control.
Edit Submit #2: A direct imporvement done was to format the results of the queries to be displayed in a more user friendly way such as DataFrame with column headers.


- Also I believe there are more quality control to be added such as number of rows in the files against rows inserted in each tables.
- Same goes for making sure each table is consistent and as the same number of rows.
- One other improvement would be to try to reduce the number of tables to 2, without impacting the performances and delivering the same queries.
