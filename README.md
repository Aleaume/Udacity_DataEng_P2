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



## Database Creation & Queries

### Database

### Table Creation

### Queries

## ETL (Extract Transform Load)

## Improvement suggestions / Additional work
