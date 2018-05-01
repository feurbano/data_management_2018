# Dealing with Spatio-temporal Data in Movement and Population Ecology: PROGRAM

## 0. Introduction to Data Management (Cagnacci & Urbano, 2 h)
### 0.1 BASIC CONCEPTS
* 0.1.1 Introduction to spatial objects in animal ecology [Cagnacci, 1 hour]
* 0.1.2 Introduction to Data Management and Spatial Database [Urbano, 30 minutes] 
 
### 0.2 PRELIMINARY STEPS
* 0.2.1 Installation of PostgreSQL/PostGIS and PgAdmin [?, 0.5 hours]

## 1. SQL and Spatial SQL (Urbano + ?, 6 h)
### 1.1 WORKING WITH SQL 
* 1.1.1 Introduction to SQL
* 1.1.2 Overview of the database used for the exercises
* 1.1.3 Schemas, tables, data types
* 1.1.4 SELECT, FROM, WHERE
* 1.1.5 AND, OR, IN, !=, NULL
* 1.1.6 ORDER BY and LIMIT
* 1.1.7 LIKE
* 1.1.8 GROUP BY (COUNT, SUM, MIN, MAX, AVG, STDDEV)
* 1.1.9 HAVING
* 1.1.10 Joining multiple tables
* 1.1.11 LEFT JOIN
* 1.1.12 Nested queries
* 1.1.13 WINDOW functions

### 1.2 WORKING WITH SPATIO-TEMPORAL SQL
* 1.2.1 Temporal data (date, time, timezone), EXTRACT
* 1.2.2 Spatial objects in PostGIS (points, lines, polygons, raster)
* 1.2.3 Visualize the coordinates of a spatial object
* 1.2.4 Reference systems and projections 
* 1.2.5 Create a point from coordinates
* 1.2.6 Create a line from ordered points (trajectory)
* 1.2.7 Calculate length of a trajectory
* 1.2.8 Create a polygon from points (convex hull)
* 1.2.9 Calculate the area of a polygon
* 1.2.10 Visualize spatial data in QGIS

## 2. Population Ecology Data Management (Urbano, 6 h)
### 2.1 SETTING UP THE POPULATION ECOLOGY DATABASE 
* 2.1.1 Introduction to the goals and the data sets used for the study case on Population Ecology Data
* 2.1.2 From Field Data to a Spatial Database: how to deal with data
* 2.1.3 Review of the raw data and identification of errors and outliers in the original spreadsheets
* 2.1.4 Move to a database: database data model
* 2.1.5 Create tables and primary keys to store data 
* 2.1.6 Create temporary tables to import data (check and fix errors in data)
* 2.1.7 Dealing with missing, suspicious and wrong information
* 2.1.8 Create external keys and look up tables (check and fix errors in data)
* 2.1.9 Create spatial objects
* 2.1.10 Create view for users and applications
 
### 2.2 POPULATION ECOLOGY: EXERCISES WITH SQL AND SPATIAL SQL 
* 2.2.1 Exercise 1: ... (SQL)
* 2.2.2 Exercise 2: ... (SQL)
* 2.2.3 Exercise 3: ... (Spatial SQL)

## 3. Movement Ecology Data Management (Urbano, 6 h)
### 3.1 SETTING UP THE MOVEMENT ECOLOGY DATABASE
* 3.1.1 Introduction to the goals and the data sets used for the study case on Movement Ecology Data
* 3.1.2 Managing and modelling information on animals and sensors
* 3.1.3 Import data into the database
* 3.1.4 Create keys, indices and domains
* 3.1.5 From data to information: associating locations to animals
* 3.1.6 Manage the location data in a spatial database
* 3.1.7 From locations to trajectories and home ranges
* 3.1.8 Integrating spatial ancillary information: land cover
* 3.1.9 Data quality: how to detect and manage outliers
* 3.1.10 Raster Data in PostGIS (demo)

### 3.2 MOVEMENT ECOLOGY: EXERCISES WITH SQL AND SPATIAL SQL 
* 3.2.1 Exercise 1: Calculate number of locations per animal per month
* 3.2.2 Exercise 2: Calculate average distance per animal per month
* 3.2.3 Exercise 3: Find animals at same place at the same time
* 3.2.4 Exercise 4: Find locations of an animal in the home range (convex hulls) of another animal
* 3.2.5 Exercise 5: Calculate percentage of location per land cover type per animal


## 4. Introductions to Data analysis in R (Basille, 2 h)
### 4.1 R FOR POPULATION AND MOVEMENT ECOLOGY DATA
* 4.1.1 Introduction to R
* 4.1.2 Relevant R packages

### 4.2 INTEGRATION OF SPATIAL DB WITH R 
* 4.2.1 How to Harmonize Software Environments

## 5. Population Ecology Data Analysis (Basille, 2 h)
### 5.1 FIRST SCIENTIFIC QUESTION 
* 5.1.1 Analysis step 1 [...]
* 5.1.2 Analysis step 2 [...]
* 
### 5.2 SECOND SCIENTIFIC QUESTION 
* 5.2.1 Analysis step 1 [...]
* 5.2.2 Analysis step 2 [...]

## 6. Movement Ecology Data Analysis (4 h)
### 6.1 FIRST SCIENTIFIC QUESTION 
* 6.1.1 Analysis step 1 [...]
* 6.1.2 Analysis step 2 [...]

### 6.2 SECOND SCIENTIFIC QUESTION
* 6.2.1 Analysis step 1 [...]
* 6.7.2 Analysis step 2 [...]

## 7. Resource Selection Analysis in Movement and Population Ecology (Van Loon, 4 h)
### 7.1 THEORY [...] 
* 7.1.1 Step 1
* 7.1.2 Step 2

### 7.2 EXERCISE [...] 
* 7.2.1 Step 1
* 7.2.2 Step 2

## 8. Special topics
### 8.1 PRESENTATIONS (2 h)
* 8.1.1 The Ecological Context Built from Satellites, including Sentinel [Rocchini, 30 minutes]
* 8.1.2 Dealing with Acceleration Data [Berger, 30 minutes]
* 8.1.3 Data sharing and Data standards for a better Science [Davidson, 30 minutes]
* 8.1.4 From population data to spatial modelling [Nilsen, 30 minutes]
