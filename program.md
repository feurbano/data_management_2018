# Dealing with Spatio-Temporal Data in Movement and Population Ecology: PROGRAM

## 0. Introduction to Data Management (Cagnacci & Urbano, 2.5 h)
### 0.1 BASIC CONCEPTS
* 0.1.1 Introduction to spatial objects in animal ecology 
* 0.1.2 Introduction to Data Management and Spatial Database  
 
### 0.2 PRELIMINARY STEPS
* 0.2.1 Installation of PostgreSQL/PostGIS and pgAdmin 
* 0.2.2 Connection to the database tracking_db
* 0.2.3 Exploration of pgAdmin Interface
 
## 1. SQL and Spatial SQL (Urbano, 6 h)
### 1.1 WORKING WITH SQL 
* 1.1.1 Introduction to SQL
* 1.1.2 Overview of the database used for the exercises
* 1.1.3 Schemas, tables, data types
* 1.1.4 SELECT, FROM, WHERE
* 1.1.5 AND, OR, IN, !=, NULL
* 1.1.6 ORDER BY, LIMIT, DISTINCT
* 1.1.7 LIKE
* 1.1.8 GROUP BY (COUNT, SUM, MIN, MAX, AVG, STDDEV)
* 1.1.9 HAVING
* 1.1.10 Joining multiple tables
* 1.1.11 LEFT JOIN
* 1.1.12 Nested queries
* 1.1.13 WINDOW functions
* 1.1.14 INSERT, UPDATE, DELETE

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

## 2. Movement Ecology Data Management (Urbano, 10 h)
### 2.1 SETTING UP THE MOVEMENT ECOLOGY DATABASE
* 2.1.1 Introduction to the goals and the datasets
* 2.1.2 Create a db and import sensor data
* 2.1.3 Create acquisition timestamps, indexes and permissions
* 2.1.4 Managing and modelling information on animals and sensors 
* 2.1.5 From data to information: associating locations to animals
* 2.1.6 Manage the location data in a spatial database
* 2.1.7 From locations to trajectories and home ranges
* 2.1.8 Integrating spatial ancillary information: land cover
* 2.1.9 Data quality: how to detect and manage outliers
* 2.1.10 Data export and maintenance
* 2.1.11 Database maintenance
* 2.1.12 Raster Data in PostGIS (demo)
* 2.1.13 Deal with data collected on the field (demo)

### 2.2 EXERCISES WITH SQL AND SPATIAL SQL 
* 2.2.1 Exercise 1: Calculate number of locations per animal per month
* 2.2.2 Exercise 2: Calculate average distance per animal per month
* 2.2.3 Exercise 3: Find animals at same place at the same time
* 2.2.4 Exercise 4: Find locations of an animal in the home range (convex hulls) of another animal
* 2.2.5 Exercise 5: Calculate percentage of location per land cover type per animal

## 3. Movement Ecology Data Analysis with R (Basille, 6 h)
### 3.1 R FOR POPULATION AND MOVEMENT ECOLOGY DATA
* 3.1.1 Introduction to R
* 3.1.2 Relevant R packages
* 3.1.3 How to Harmonize Software Environments

### 3.2 DATA ANALYSIS: (QUESTION 1)
* 3.2.1 Analysis step 1 (...)
* 3.2.2 Analysis step 2 (...)
* 3.2.n Analysis step n (...)

### 3.3 DATA ANALYSIS: (QUESTION 2)
* 3.3.1 Analysis step 1 (...)
* 3.3.2 Analysis step 2 (...)
* 3.3.n Analysis step n (...)

## 4. From Population Data to Spatial Modelling (Nilsen, 4 h)
### 4.1 THEORY (...)
* 4.1.1 Step 1
* 4.1.2 Step 2

### 4.2 EXERCISE (...) 
* 4.2.1 Step 1
* 4.2.2 Step 2

## 5. Resource Selection Analysis in Movement Ecology (Van Loon, 4 h)
### 5.1 THEORY (...)
* 5.1.1 Step 1
* 5.1.2 Step 2

### 5.2 EXERCISE (...) 
* 5.2.1 Step 1
* 5.2.2 Step 2


## 6. Special Topics
### 6.1 PRESENTATIONS (2 h)
* 6.1.1 The Ecological Context Built from Satellites, including Sentinel (Rocchini, 30 minutes)
* 6.1.2 Dealing with Acceleration Data (Berger, 30 minutes)
* 6.1.3 Data sharing and Data Standards for a Better Science (Davidson, 30 minutes)
