# 2. Movement Ecology Data Management (Urbano, 6 h)
## 2.1 SETTING UP THE MOVEMENT ECOLOGY DATABASE
In this lesson, you are guided through how to set up a new database in which you will create a table to accommodate the test GPS data sets. You create a new table in a dedicated schema. This lesson describes how to upload the raw GPS data coming from five sensors deployed on roe deer in the Italian Alps into the database and how tocreate additional database users.

The datasets you will need for the following lessons can be downloaded **[here](https://github.com/feurbano/data_management_2018/tree/master/data/tracking_db.zip)**. Download and unzip the contents to your computer. If using Windows, unzip them to a location the `postgres` database user can access (e.g., a new folder under `C:/tracking_db/`).

### 2.1.1 Introduction to the goals and the data set
Once a tracking project starts and sensors are deployed on animals, data begin to arrive (usually in the form of text files containing the raw information recorded by sensors). At this point, data must be handled by researchers. The aim of this exercise is to set up an operational database where GPS data coming from roe deer monitored in the Alps can be stored, managed and analysed. The information form the sensors must be complemented with other information on the individuals, deployments and surrounding environment to have a complete picture of the animals movement.


### 2.1.2 Create a db and import sensor data

Assuming that you have PostgreSQL installed and running on your computer (or server), the first thing that you have to do to import your raw sensors data into the database is to connect to the database server and create a new database with the command **[CREATE DATABASE](http://www.postgresql.org/docs/devel/static/sql-createdatabase.html)**:

```sql
CREATE DATABASE gps_tracking_db
ENCODING = 'UTF8'
TEMPLATE = template0
LC_COLLATE = 'C'
LC_CTYPE = 'C';
```

You could create the database using just the first line of the code. The other lines are added just to be sure that the database will use UTF8 as encoding system and will not be based on any local-specific setting regarding, e.g. alphabets, sorting, or number formatting. This is very important when you work in an international environment where different languages (and therefore characters) can potentially be used. When you import textual data with different encoding, you have to specify the original encoding otherwise special character might be misinterpreted. Different encodings are a typical source of error (e.g. character with accents that are transformed into strange symbols) when data are moved from a system to another.

Although not compulsory, it is very important to document the objects that you create in your database to enable other users (and probably yourself later on) to understand its structure and content, pretty much the same way you use to do with metadata of your data. **[COMMENT](http://www.postgresql.org/docs/devel/static/sql-comment.html)** command gives you this possibility. Comments are stored into the database. In this case:

```sql
COMMENT ON DATABASE gps_tracking_db   
IS 'Next Generation Data Management in Movement Ecology Summer school: my database.'; 
```

By default, a database comes with the *public* schema; it is good practice, however, to use different schemas to store user data. For this reason you create a new **[SCHEMA](http://www.postgresql.org/docs/devel/static/sql-createschema.html)** called *main*:

```sql
CREATE SCHEMA main;
```

```sql
COMMENT ON SCHEMA main IS 'Schema that stores all the GPS tracking core data.'; 
```

Before importing the GPS data sets into the database, it is recommended that you examine the source data (usually .dbf, .csv, or .txt files) with a spreadsheet or a text editor to see what information is contained. Every GPS brand/model can produce different information, or at least organise this information in a different way, as unfortunately no consolidated standards exist yet. The idea is to import raw data (as they are when received from the sensors) into the database and then process them to transform data into information. Once you identify which attributes are stored in the original files (for example, *GSM01438.csv*), you can create the structure of a table with the same columns, with the correct database **[DATA TYPES](http://www.postgresql.org/docs/devel/static/datatype.html)**. You can find the GPS data sets in .csv files included in the *tracking_DB.zip* file with test data in the sub-folder */tracking\_db/data/sensors\_data*. We have to write the SQL code that generates the same table structure of the source files (Vectronic GPS collars) within the database. The new table is called here *main.gps\_data* (*main* is the name of the schema where the table will be created, while *gps\_data* is the name of the table). The SQL code is

```sql
CREATE TABLE main.gps_data 
( 
gps_data_id serial NOT NULL, 
gps_sensors_code character varying, 
line_no integer, 
utc_date date, 
utc_time time without time zone, 
lmt_date date, 
lmt_time time without time zone, 
ecef_x integer, 
ecef_y integer, 
ecef_z integer, 
latitude double precision, 
longitude double precision, 
height double precision, 
dop double precision, 
nav character varying(2), 
validated character varying(3), 
sats_used integer, 
ch01_sat_id integer, 
ch01_sat_cnr integer, 
ch02_sat_id integer, 
ch02_sat_cnr integer, 
ch03_sat_id integer, 
ch03_sat_cnr integer, 
ch04_sat_id integer, 
ch04_sat_cnr integer, 
ch05_sat_id integer, 
ch05_sat_cnr integer, 
ch06_sat_id integer, 
ch06_sat_cnr integer, 
ch07_sat_id integer, 
ch07_sat_cnr integer, 
ch08_sat_id integer, 
ch08_sat_cnr integer, 
ch09_sat_id integer, 
ch09_sat_cnr integer, 
ch10_sat_id integer, 
ch10_sat_cnr integer, 
ch11_sat_id integer, 
ch11_sat_cnr integer, 
ch12_sat_id integer, 
ch12_sat_cnr integer, 
main_vol double precision, 
bu_vol double precision, 
temp double precision, 
easting integer, 
northing integer, 
remarks character varying 
); 
```

```sql
COMMENT ON TABLE main.gps_data 
IS 'Table that stores raw data as they come from the sensors (plus the ID of the sensor).'; 
```

In a relational database, each table must have a primary key, that is a field (or combination of fields) that uniquely identify each record. In this case, as no field represents a unique value in the dataset, we add a **[SERIAL](http://www.postgresql.org/docs/devel/static/sql-createsequence.html)** id field managed by the database (as a sequence of integers automatically generated) to be sure that it is unique. We set this field as the primary key of the table:

```sql
ALTER TABLE main.gps_data 
ADD CONSTRAINT gps_data_pkey PRIMARY KEY(gps_data_id); 
```

To keep track of database changes, it is useful to add another field where the timestamp of the insert of each record is automatically recorded (assigning a dynamic default value):

```sql
ALTER TABLE main.gps_data ADD COLUMN insert_timestamp timestamp with time zone; 
```

```sql
ALTER TABLE main.gps_data ALTER COLUMN insert_timestamp SET DEFAULT now(); 
```

Now we are ready to import our data sets. There are many ways to do so. The main one is to use the **[COPY](http://www.postgresql.org/docs/devel/static/sql-copy.html)** command setting the appropriate parameters:

```sql
COPY main.gps_data( 
gps_sensors_code, line_no, utc_date, utc_time, lmt_date, lmt_time, ecef_x, ecef_y, ecef_z, latitude, longitude, height, dop, nav, validated, sats_used, ch01_sat_id, ch01_sat_cnr, ch02_sat_id, ch02_sat_cnr, ch03_sat_id, ch03_sat_cnr, ch04_sat_id, ch04_sat_cnr, ch05_sat_id, ch05_sat_cnr, ch06_sat_id, ch06_sat_cnr, ch07_sat_id, ch07_sat_cnr, ch08_sat_id, ch08_sat_cnr, ch09_sat_id, ch09_sat_cnr, ch10_sat_id, ch10_sat_cnr, ch11_sat_id, ch11_sat_cnr, ch12_sat_id, ch12_sat_cnr, main_vol, bu_vol, temp, easting, northing, remarks) 
FROM 
'C:\tracking_db\data\sensors_data\GSM01438.csv' WITH CSV HEADER DELIMITER ';'; 
```

You might have to adapt the path to the file. If you get an error about permissions (you cannot open the source file) it means that the file is stored in a folder where it is not accessible by the database. In this case, move it to a folder that is accessible to all users of your computer.

If PostgreSQL complain that date is out of range, check the standard date format used by PostgreSQL (**[datestyle](http://www.postgresql.org/docs/devel/static/runtime-config-client.html#GUC-DATESTYLE)**):

```sql
SHOW datestyle; 
```

In the original file, date is expressed as DD/MM/YY, so if it is not set to *ISO, DMY*, then you have to set the date format (for the current session) as:

```sql
SET SESSION datestyle = "ISO, DMY"; 
```

This will change the *datastyle* for the current session only. If you want to change this setting permanently, you have to modify the *datestyle* option in the **[postgresql.conf](http://www.postgresql.org/docs/devel/static/config-setting.html#CONFIG-SETTING-CONFIGURATION-FILE)** file.

PgAdmin offers the possibility to import data (including local data to a server) with a graphical interface (right click on the table, and select *Import* setting all the proper parameters and options). 

#### Exercise

Import into the database the raw GPS data from the test sensors:

-   GSM01508
-   GSM01511
-   GSM01512


### 2.1.3 Create acquisition timestamps, indexes and permissions

#### Acquisition timestamps
In the original GPS data file, no timestamp field is present. Date and time are kept in two separate fields, but the moment in time they identify is given by the combination of the two. To properly deal with this information, the two piece of information must be combined in a single field where also the correct time zone is set (in this case UTC, but in other case the time zone is not explicitly defined in the field name and you have to check in the documentation of your GPS sensor). Although the table main.gps\_data is designed to store data as they come from the sensors, it is convenient to have this field in the table. To do so, you first add a field with data type *timestamp with time zone* type. Then you fill it (with an UPDATE statement) from the time and date fields. 

```sql
ALTER TABLE main.gps_data 
  ADD COLUMN acquisition_time timestamp with time zone;
```

```sql
UPDATE main.gps_data 
  SET acquisition_time = (utc_date + utc_time) AT TIME ZONE 'UTC';
```

#### Exercise
* Find the temporal duration of the dataset for each animal
*Hint: get the minimum and maximum value of acquisition time for each animal and then make the difference to have the interval*

* Retrieve data from the collar **GSM01512** during the month of May (whatever the year), and order them by their acquisition time
*Hint: use extract (month ...) to set the criteria on month*

#### Indexes

A database offers many functionalities to improve performances, that is a key element when very large data sets have to be manipulated. One of these are **[indexes](http://www.postgresql.org/docs/devel/static/indexes.html)** that are data structures that improve the speed of data retrieval operations on a database table at the cost of slower writes and the use of more storage space. Database indexes work in a similar way to a book's table of contents: you have to add an extra page and update it whenever new content is added, but then searching for specific sections will be much faster. For example, if you look for records related to a specific sensor in our *main.gps_data* table, with no advance preparation, the system would have to scan the entire table, row by row, to find all matching entries. Since the table can potentially contain many rows but only a fraction are related to one specific animal, this is clearly an inefficient method for retrieving the records. But if the system has been instructed to maintain an index on the *gps_sensors_code* column, it can use a more efficient method for locating matching rows. For instance, it might only have to walk a few levels deep into a search tree.

Once an index is created, no further intervention is required: the system will update the index when the table is modified, and it will use the index in queries when it thinks doing so would be more efficient than a sequential table scan. But you might have to run the **[ANALYZE](http://www.postgresql.org/docs/current/static/sql-analyze.html)** command regularly to update statistics to allow the query planner to make educated decisions.

After an index is created, the system has to keep it synchronized with the table. This adds overhead to data manipulation operations. Therefore indexes should be created only for those fields that might be used as selection criteria in queries. 

In the table *main.gps_data* two strong candidates are *acquisition\_time* and the *gps\_sensors\_code* fields, which are probably two key attributes in the retrieval of data
from this table:

```sql
CREATE INDEX acquisition_time_index
  ON main.gps_data
  USING btree (acquisition_time );
```

```sql
CREATE INDEX gps_sensors_code_index
  ON main.gps_data
  USING btree (gps_sensors_code);
```

#### Permissions

One of the main advantages of an advanced database management system like PostgreSQL is that the database can be accessed by a number of different users at the same time, keeping the data always in a single version with a proper management of concurrency. This ensures that the database maintains the ACID (atomicity, consistency, isolation, durability) principles in an efficient manner and that different permission levels can be set for each user. For example, you can have a single administrator that can change the database, a set of advanced users that can edit the content of the core tables and create their own object (e.g. tables, functions) without changing the main database structure, and a set of users that can just read the data.

In PostgreSQL, all the permissions are related to the concept of a `ROLE`. Roles can own database objects (for example, tables) and can assign privileges on those objects to other roles to control who has access to which objects. Furthermore, it is possible to grant membership in a role to another role, thus allowing the member role to use privileges assigned to another role (For example you assign permission to a generic "student" group and then you add students to the group that automatically inherit all the permission of the group).

Every connection to the database server is made using the name of some particular role, and this role determines the initial access privileges for commands issued in that connection: database users are just `ROLE` with the `LOGIN`privilege. T

Now you create a new group of database users and related permissions (**[roles and privileges](http://www.postgresql.org/docs/9.0/static/user-manag.html)**) on the raw locations table to illustrate some of the options to manage different class of users.

In most of the cases, your database will have multiple users. As an example, you create here a group of users, called *basic\_users*.

```sql
CREATE ROLE basic_user LOGIN
NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;
```

You can then associate a permission to *SELECT* data from the raw locations data. First, you have to give USAGE permission on the SCHEMA where the *main.gps\_data* table is stored (with `SET USAGE ON THE SCHEMA`), then you grant READ permission on the table.

```sql
GRANT USAGE ON SCHEMA main TO basic_user;
```

```sql
GRANT SELECT ON main.gps_data TO basic_user;
```

As mentioned before, groups are very useful because you can associate multiple users to the same group and they will automatically inherit all the permissions of the group so you do not have to assign permissions to each one individually. Permissions can be given to a whole group or to specific users.

You create a user that is part of the *basic\_users* group (login *user1*, password *user1\_password*).

```sql
CREATE ROLE user1 LOGIN IN ROLE basic_user
  PASSWORD 'user1_password'
  NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;
```

In general, every time you create a new database object, you have to set the privileges (for tables: `ALL`, `INSERT`, `SELECT`, `DELETE`, `UPDATE`) to groups/users. If you want to set the same privileges to a group/user for all the tables in a specific schema you can use GRANT SELECT ON ALL TABLES:

```sql
GRANT SELECT ON ALL TABLES 
  IN SCHEMA main 
  TO basic_user;
```

You can now reconnect to the database as *user1* user and try first to visualize data and then change the values of a record. This latter operation will be forbidden because the user do not have this kind of privilege. In this way, the database integrity will be safeguarded from inexperienced user that might remove all the data running the wrong command.

If you want to automatically grant permissions to specific groups/users to all new (i.e. that will be created in the future) objects in a schema, you can use ALTER DEFAULT PRIVILEGES:

```sql
ALTER DEFAULT PRIVILEGES 
  IN SCHEMA main 
  GRANT SELECT ON TABLES 
  TO basic_user;
```

From now on, users belonging to *basic\_user's* group with have reading access to all the tables that will be created in the *main* schema. By default, all objects created in the schema *public* are fully accessible to all users.

### 2.1.4 Managing and modelling information on animals and sensors
### 2.1.5 From data to information: associating locations to animals
### 2.1.6 Manage the location data in a spatial database
### 2.1.7 From locations to trajectories and home ranges
### 2.1.8 Integrating spatial ancillary information: land cover
### 2.1.9 Data quality: how to detect and manage outliers

### 2.1.10 Data export 
There are different ways to export a table or the results of a query to an external file. One is to use the command [COPY (TO)](http://www.postgresql.org/docs/devel/static/sql-copy.html). `COPY TO` (similarly to what happens with the command `COPY FROM` used to import data) with a file name directly write the content of a table or the result of a query to a file, for example in .csv format. The file must be accessible by the PostgreSQL user (i.e. you have to check the permission on target folder by the user ID the PostgreSQL server runs as) and the name (path) must be specified from the viewpoint of the server. This means that files can be read or write only in folders 'visible' to the database servers. If you want to remotely connect to the database and save data into your local machine, you should use the command [\COPY](http://www.postgresql.org/docs/devel/static/app-psql.html#APP-PSQL-META-COMMANDS-COPY) instead. It performs a frontend (client) copy. `\COPY` is not an SQL command and must be run from a PostgreSQL interactive terminal [PSQL](http://www.postgresql.org/docs/devel/static/app-psql.html). This is an operation that runs an SQL COPY command, but instead of the server reading or writing the specified file, PSQL reads or writes the file and routes the data between the server and the local file system. This means that file accessibility and privileges are those of the local user, not the server, and no SQL superuser privileges are required. 
Another possibility to export data is to use the pgAdmin interface: in the SQL console select `Query/Execute to file`, the results will be saved to a local file instead of being visualized. Other database interfaces have similar tools. This can be applied to any query.
For spatial data, the easiest option is to load the data in QGIS and then save as shapefile (or any other format) on your computer.

### 2.1.11 Database maintenance
Once your database is populated and used for daily work, it is a *really* good idea to routinely make a safe copy of your data. Since the RDBMS maintains data in a binary format which is not meant to be tampered with, we need to `dump` the database content in a format suitable for being later restored if it needs be. The very same dump could also be used for replicating the database contents on another server.
From pgAdmin, the operation of making a database dump is extremely simple: right click the database and choose `Backup`.
There are a few output formats, apart from the default `Custom` one. With `Plain` the file will be plain (readable) SQL commands that can be opened (and edit, if needed) with a text editor (e.g. Notepad++). `Tar` will generate a compressed file that is convenient if you have frequent backups and you want to maintain an archive. For more info, see the docs on [backup and restore](http://www.postgresql.org/docs/current/static/backup-dump.html) for further information.
If you want to automatically generate a backup of your database, you can create a bash script and schedule its execution on your server with the desired frequency. Here an example on a Windows server:

```
@echo off
   for /f "tokens=1-3 delims=/ " %%i in ("%date%") do (
     set day=%%i
     set month=%%j
     set year=%%k
   )
   set datestr=%year%%month%%day%

   echo on
   
   C:/PostgreSQL/9.5/bin/pg_dump.exe --host localhost --port 5432 --username "postgres" --no-password  --format tar --blobs --encoding UTF8 --verbose --file "C:/backup/bu_dbtracking_%datestr%.backup"  -d tracking_db --schema "lu_tables" --schema "main" --schema "tools" 
```

In this case, the name of the file generated includes the current date. Only three schema (main, lu_tables and tools) are backed up.

If you have tables that change frequently and others that remain unchanged for long periods of time, you can plan frequent backups for the former and occasional backups for the latter.

### 2.1.12 Raster data in PostGIS (demo)
The advancement in movement ecology from a data perspective can reach its full potential only by combining the technology of animal tracking with the technology of other environmental sensing programmes. Ecology is fundamentally spatial, and animal ecology is obviously no exception. Any scientific question in animal ecology cannot overlook the dynamic interaction between individual animals or populations, and the environment in which the ecological processes occur. Movement provides the mechanistic link to explain this complex ecosystem interaction, as the movement path is dynamically determined by external factors, through their effect on the individual's state and the life-history characteristics of an animal. Therefore, most modelling approaches for animal movement include environmental factors as explanatory variables.  

> RASTER IN POSTGIS

#### DEMONSTRATION 1: Analyzing movement data with a (raster) environmental layer
In these examples we will explore some simple analysis performed with spatial SQL into our GPS tracking with **land cover/use data** derived from [CORINE land cover database](https://land.copernicus.eu/pan-european/corine-land-cover) (as a static raster layer). 
##### Set up raster layer into the database

Import land cover layer (CORINE data set) *(only example, not run)*

`raster2pgsql.exe -C -t 128x128 -M -r C:/tracking_db/data/env_data/raster/corine_land_cover_2006.tif env_data.land_cover | psql.exe -d eurodeer_db -U postgres -p 5432`

Meaning of raster2pgsql parameters:
* -C: new table
* -t: divide the images in tiles
* -M: vacuum analyze the raster table
* -r: Set the constraints for regular blocking

##### Create a table for land cover raster data from an existing (larger) DB layer (clip)
```sql
CREATE TABLE env_data.land_cover (rid SERIAL primary key, rast raster);

CREATE INDEX land_cover_rast_idx 
  ON env_data.land_cover 
  USING GIST (ST_ConvexHull(rast));
```
```sql
INSERT INTO env_data.land_cover (rast)
SELECT 
  rast
FROM 
  env_data.corine_land_cover_2006, 
  main.study_areas
WHERE 
  st_intersects(rast, ST_Expand(st_transform(geom, 3035), 5000)) AND 
  study_areas_id = 1;
```
```sql
SELECT AddRasterConstraints('env_data'::name, 'land_cover'::NAME, 'rast'::name);
```

##### Export the layer to tiff
Create a new table with all raster unioned, add constraints, export to TIFF with GDAL, drop the table
```sql
CREATE TABLE env_data.land_cover_export(rast raster);
```
```sql
INSERT INTO 
  env_data.land_cover_export
SELECT 
  st_union(rast) AS rast 
FROM 
  env_data.land_cover;
```
```sql
SELECT AddRasterConstraints('env_data'::name, 'land_cover_export'::name, 'rast'::name);
```
Export with GDAL_translate  

`gdal_translate -of GTIFF "PG:host=eurodeer2.fmach.it dbname=eurodeer_db user='postgres' schema=env_data table=land_cover_export mode=2" C:\Users\User\Desktop\landcover\land_cover.tif`

Remove the unioned table
```sql
DROP TABLE env_data.land_cover_export;
```

##### Intersect the fixes with the land cover layer for the animal 782
```sql
SELECT  
  st_value(rast,st_transform(geom, 3035)) as lc_id
FROM 
  env_data.gps_data_animals,
  env_data.land_cover
WHERE
  animals_id = 782 AND
  gps_validity_code = 1 AND
  st_intersects(st_transform(geom, 3035), rast);
```
##### Calculate the percentage of each land cover class for fixes of the animal 782
```sql
WITH locations_landcover AS 
(
SELECT  
  st_value(rast,st_transform(geom, 3035)) AS lc_id
FROM 
  env_data.gps_data_animals,
  env_data.land_cover
 WHERE
  animals_id = 782 AND
  gps_validity_code = 1 AND
  st_intersects(st_transform(geom, 3035), rast)
)
SELECT
  lc_id,
  label3,
  (count(*) * 1.0 / (SELECT count(*) FROM locations_landcover))::numeric(5,4) AS percentage
FROM 
  locations_landcover,
  env_data.corine_land_cover_legend
WHERE
  grid_code = lc_id
GROUP BY 
  lc_id,
  label3
ORDER BY
  percentage DESC;
```

##### Intersect the convex hull of animal 782 with the land cover layer
```sql
SELECT 
  (stats).value AS grid_code, 
  (stats).count AS num_pixels
FROM 
  (
  SELECT
    ST_valuecount(ST_union(st_clip(rast ,st_transform(geom,3035)))) AS stats
  FROM
    env_data.view_convexhull,
    env_data.land_cover
  WHERE
    animals_id = 782 AND
    st_intersects (rast, st_transform(geom,3035))
  ) a
```

##### Calculate the percentage of each land cover class in the convex hull for the animal 782
```sql
WITH convexhull_landcover AS 
(
SELECT 
  (stats).value AS lc_id, 
  (stats).count AS num_pixels
FROM 
  (
  SELECT
    ST_valuecount(ST_union(st_clip(rast ,st_transform(geom,3035))))  stats
  FROM
    env_data.view_convexhull,
    env_data.land_cover
  WHERE
    animals_id = 782 AND
    st_intersects (rast, st_transform(geom,3035))
  ) AS a
)
SELECT
  lc_id,
  label3,
  (num_pixels * 1.0 / (sum(num_pixels)over()))::numeric(5,4) AS percentage
FROM 
  convexhull_landcover,
  env_data.corine_land_cover_legend
WHERE
  grid_code = lc_id
ORDER BY
  percentage DESC;
```

##### Intersect the fixes for males vs female with the land cover layer
```sql
SELECT
  sex,  
  ST_Value(rast, ST_Transform(geom, 3035)) AS lc_id,
  count(*) AS number_locations
FROM 
  env_data.gps_data_animals,
  env_data.land_cover,
  main.animals
WHERE
  animals.animals_id = gps_data_animals.animals_id AND
  gps_validity_code = 1 AND
  ST_Intersects(ST_Transform(geom, 3035), rast)
GROUP BY 
  sex, lc_id
ORDER BY 
  lc_id;
```

##### Calculate the percentage of different land cover classes for all the monthly convex hulls of the animal 782
```sql
WITH convexhull_landcover AS
(
SELECT 
  months,
  (stats).value AS lc_id, 
  (stats).count AS num_pixels
FROM (
  SELECT 
    months, 
    ST_ValueCount(ST_Union(ST_Clip(rast ,ST_Transform(geom,3035))))  stats
  FROM
    env_data.view_convexhull_monthly,
    env_data.land_cover
  WHERE
    ST_Intersects (rast, ST_Transform(geom,3035))
  GROUP BY 
    months) a
)
SELECT
  months,
  label3,
  (num_pixels * 1.0 / (sum(num_pixels) over (PARTITION BY months)))::numeric(5,4) AS percentage
FROM 
  convexhull_landcover,
  env_data.corine_land_cover_legend
WHERE
  grid_code = lc_id
ORDER BY
  label3, months;
```

##### Calculate the percentage of each land cover class for male/female *(takes a bit)*
```sql
WITH locations_landcover AS
(
SELECT
  sex,  
  st_value(rast,st_transform(geom, 3035)) AS lc_id,
  count(*) AS number_locations
FROM 
  env_data.gps_data_animals,
  env_data.land_cover,
  main.animals
 WHERE
  animals.animals_id = gps_data_animals.animals_id AND
  gps_validity_code = 1 AND
  st_intersects(st_transform(geom, 3035), rast)
GROUP BY sex, lc_id
) 
SELECT
  sex,
  label3,
  (number_locations *1.0 / sum(number_locations) OVER (partition by sex))::numeric(5,4) AS percentage
FROM 
  locations_landcover,
  env_data.corine_land_cover_legend
WHERE
  grid_code = lc_id 
ORDER BY
  label3, sex;
```

#### DEMONSTRATION 2: Analyzing location data with a time series of environmental layers

Animal locations are not only spatial, but are fully defined by spatial and temporal coordinates (as given by the acquisition time). Logically, the same temporal definition also applies to environmental layers. Some characteristics of the landscape, such as land cover or road networks, can be considered static over a large period of time and these environmental layers are commonly intersected with animal locations to infer habitat use and selection by animals. However, many characteristics actually relevant to wildlife, such as vegetation biomass or road traffic, are indeed subject to temporal variability (on the order of hours to weeks) in the landscape, and would be better represented by dynamic layers that correspond closely to the conditions actually encountered by an animal moving across the landscape. Nowadays, satellite-based remote sensing can provide high temporal resolution global coverage of medium/high-resolution images that can be used to compute a large number of environmental parameters very useful to wildlife studies. One of the most common set of environmental data time series is the Normalized Difference Vegetation Index (NDVI), but other examples include data sets on snow, ocean primary productivity, surface temperature, or salinity. Snow cover, NDVI, and sea surface temperature are some examples of indexes that can be used as explanatory variables in statistical models or to parametrize bayesian inferences or mechanistic models. The main shortcoming of such remote-sensing layers is the relatively low spatial and/or temporal resolution, which does not fit the current average bias of wildlife-tracking GPS locations (less than 20 m) and temporal scale of animal movement, thus potentially leading to a mismatch between the animal-based information and the environmental layers (note that the resolution can still be perfectly fine, depending on the overall spatial and temporal variability and the species and biological process under study). Higher-resolution images and new types of information (e.g. forest structure) are presently provided by new types of sensors, such as those from lidar, radar, or hyper-spectral remote-sensing technology and Sentinel 2 (optical data). The new generation of satellites requires dedicated storage and analysis tools (e.g. Goggle Earth Engine) that can be related to the Big Data framework. 
Here, we will explore some simple example of spatio-temporal analyses that involve the interaction between GPS data and NDVI time series.

The MODIS (Moderate Resolution Imaging Spectroradiometer) instrument operates on the NASA's Terra and Aqua spacecraft. The instrument views the entire earth surface every 1 to 2 days, captures data in 36 spectral bands ranging in wavelength from 0.4 μm to 14.4 μm and at varying spatial resolutions (250 m, 500 m and 1 km). The Global MODIS vegetation indices (code MOD13Q1) are designed to provide consistent spatial and temporal comparisons of vegetation conditions. Red and near-infrared reflectances, centred at 645 nm and 858 nm, respectively, are used to determine the daily vegetation indices, including the well known NDVI. This index is calculated by contrasting intense chlorophyll pigment absorption in the red against the high reflectance of leaf mesophyll in the near infrared. It is a proxy of plant photosynthetic activity and has been found to be highly related to green leaf area index (LAI) and to the fraction of photosynthetically active radiation absorbed by vegetation (FAPAR). Past studies have demonstrated the potential of using NDVI data to study vegetation dynamics. More recently, several applications have been developed using MODIS NDVI data such as land-cover change detection, monitoring forest phenophases, modelling wheat yield, and other applications in forest and agricultural sciences. However, the utility of the MODIS NDVI data products is limited by the availability of high-quality data (e.g. cloud-free), and several processing steps are required before using the data: acquisition via web facilities, re-projection from the native sinusoidal projection to a standard latitude-longitude format, eventually the mosaicking of two or more tiles into a single tile. A number of processing techniques to 'smooth' the data and obtain a cleaned (no clouds) time series of NDVI imagery have also been implemented. These kind of processes are usually based on a set of ancillary information on the data quality of each pixel that are provided together with MODIS NDVI.

NDVI data source used in these exercises: MODIS NDVI (http://modis-land.gsfc.nasa.gov/vi.html), in a version (smoothed, weekly) downloaded from [Boku University Portal](http://ivfl-info.boku.ac.at/index.php/eo-data-processing).

##### Import MODIS NDVI time series *(only example, not run)*

`raster2pgsql.exe -C -r -t 128x128 -F -M -R -N -3000 C:/tracking_db/data/env_data/raster/MOD*.tif env_data.ndvi_modis | psql.exe -d eurodeer_db -U postgres -p 5432`

Meaning of raster2pgsql parameters
* -R: out of db raster
* -F: add a column with the name of the file
* -N: set the null value

##### Create and fill a field to explicitly mark the reference date of the images
Structure of the name of the original file: *MCD13Q1.A2005003.005.250m_7_days_NDVI.REFMIDw.tif*
```sql
ALTER TABLE env_data.ndvi_modis ADD COLUMN acquisition_date date;
UPDATE 
  env_data.ndvi_modis 
SET 
  acquisition_date = to_date(substring(filename FROM 10 FOR 7), 'YYYYDDD');
```
```sql
CREATE INDEX ndvi_modis_referemce_date_index
  ON env_data.ndvi_modis
  USING btree
  (acquisition_date);
```
##### Create a table from an existing DB layer with a larger - MODIS NDVI
```sql
CREATE TABLE env_data.modis_ndvi(
  rid serial PRIMARY KEY,
  rast raster,
  filename text,
  acquisition_date date);
```
```sql
INSERT INTO env_data.modis_ndvi (rast, filename, acquisition_date)
SELECT 
  rast, 
  filename, 
  acquisition_date
FROM
  env_data_ts.ndvi_modis_boku, 
  main.study_areas
WHERE 
  st_intersects(rast, ST_Expand(geom, 0.05)) AND 
  study_areas_id = 1;
```
```sql
SELECT AddRasterConstraints('env_data'::name, 'modis_ndvi'::NAME, 'rast'::name);
```
```sql
CREATE INDEX modis_ndvi_rast_idx 
  ON env_data.modis_ndvi
  USING GIST (ST_ConvexHull(rast));
```
```sql
CREATE INDEX modis_ndvi_referemce_date_index
  ON env_data.modis_ndvi
  USING btree
  (acquisition_date);
```

##### Extraction of a NDVI value for a point/time
```sql
WITH pointintime AS 
(
SELECT 
  ST_SetSRID(ST_MakePoint(11.1, 46.1), 4326) AS geom, 
  '2005-01-03'::date AS reference_date
)
SELECT 
  ST_Value(rast, geom) * 0.0048 -0.2 AS ndvi
FROM 
  env_data.modis_ndvi,
  pointintime
WHERE 
  ST_Intersects(geom, rast) AND
  modis_ndvi.acquisition_date = pointintime.reference_date;
```

##### Extraction of a NDVI time series of values of a given fix
```sql
SELECT 
  ST_X(geom) AS x,
  ST_Y(geom) AS y,
  acquisition_date,
  ST_Value(rast, geom) * 0.0048 -0.2 AS ndvi
FROM 
  env_data.modis_ndvi,
  env_data.gps_data_animals
WHERE 
  ST_Intersects(geom, rast) AND
  gps_data_animals_id = 1
ORDER BY 
  acquisition_date;
```

##### Extraction of the NDVI value for a fix as temporal interpolation of the 2 closest images
```sql
SELECT 
  gps_data_animals_id, 
  acquisition_time,
  DATE_TRUNC('week', acquisition_time::date)::date,
  (trunc(
    (
    ST_VALUE(pre.rast, geom) * 
    (DATE_TRUNC('week', acquisition_time::date + 7)::date - acquisition_time::date)::integer 
    +
    ST_VALUE(post.rast, geom) * 
    (acquisition_time::date - DATE_TRUNC('week', acquisition_time::date)::date))::integer/7)
    ) * 0.0048 -0.2 AS ndvi
FROM  
  env_data.gps_data_animals,
  env_data.modis_ndvi AS pre,
  env_data.modis_ndvi AS post
WHERE
  ST_INTERSECTS(geom, pre.rast) AND 
  ST_INTERSECTS(geom, post.rast) AND 
  DATE_TRUNC('week', acquisition_time::date)::date = pre.acquisition_date AND 
  DATE_TRUNC('week', acquisition_time::date + 7)::date = post.acquisition_date AND
  gps_validity_code = 1 AND
  gps_data_animals_id = 2;
```

##### Extraction of the NDVI values for a set of fixes as temporal interpolation of the 2 closest images for animal 782
```sql
SELECT 
  gps_data_animals_id, 
  ST_X(geom)::numeric (8,5) AS x,
  ST_Y(geom)::numeric (8,5) AS y,
  acquisition_time,
  DATE_TRUNC('week', acquisition_time::date)::date,
  (trunc(
    (
    ST_VALUE(pre.rast, geom) * 
    (DATE_TRUNC('week', acquisition_time::date + 7)::date - acquisition_time::date)::integer 
    +
    ST_VALUE(post.rast, geom) * 
    (acquisition_time::date - DATE_TRUNC('week', acquisition_time::date)::date))::integer/7)
    ) * 0.0048 -0.2
FROM  
  env_data.gps_data_animals,
  env_data.modis_ndvi AS pre,
  env_data.modis_ndvi AS post
WHERE
  ST_INTERSECTS(geom, pre.rast) AND 
  ST_INTERSECTS(geom, post.rast) AND 
  DATE_TRUNC('week', acquisition_time::date)::date = pre.acquisition_date AND 
  DATE_TRUNC('week', acquisition_time::date + 7)::date = post.acquisition_date AND
  gps_validity_code = 1 AND
  animals_id = 782
ORDER by 
  acquisition_time;
```

##### Calculate average, max and min NDVI for the minimum convex hull of a every month for animal 782
```sql
SELECT
  months, 
  (stats).mean  * 0.0048 - 0.2 AS ndvi_avg,
  (stats).min * 0.0048 - 0.2 AS ndvi_min,
  (stats).max * 0.0048 - 0.2 AS ndvi_max
FROM
( 
  SELECT
    months,
    ST_SummaryStats(ST_UNION(ST_CLIP(rast,geom), 'max'))  AS stats
  FROM 
    env_data.view_convexhull_monthly,
    env_data.modis_ndvi
  WHERE
    ST_INTERSECTS (rast, geom) AND 
    EXTRACT(month FROM acquisition_date) = months AND
    months IN (1,2,3)
  GROUP BY months
  ORDER BY months
) a;
```

##### Calculate time series of average, max and min NDVI for a given polygon in a given time interval
```sql 
WITH selected_area AS 
(SELECT st_setsrid(ST_MakePolygon(ST_GeomFromText('LINESTRING(11.03 45.98, 11.03 46.02, 11.08 46.02, 11.08 45.98, 11.03 45.98)')), 4326) AS geom)
SELECT
  acquisition_date, 
  ((stats).mean  * 0.0048 - 0.2)::numeric (4,3)  AS ndvi_avg,
  ((stats).min * 0.0048 - 0.2)::numeric (4,3)  AS ndvi_min,
  ((stats).max * 0.0048 - 0.2)::numeric (4,3) AS ndvi_max,
  ((stats).stddev)::numeric (6,3) AS digital_value_stddev,
  ((stats).count) AS num_pixels
FROM
( 
  SELECT
    acquisition_date,
    ST_SummaryStats(ST_UNION(ST_CLIP(rast,geom)))  AS stats
  FROM 
    selected_area,
    env_data.modis_ndvi
  WHERE
    ST_INTERSECTS (rast, geom) AND 
    acquisition_date > '1/1/2017' and acquisition_date < '30/6/2017'
  GROUP BY acquisition_date
  ORDER BY acquisition_date
) a;
```

### 2.1.13 Deal with data collected on the field (demo)

> revise and then look for a practical example

While data generated by a sensor are usually "clean" (i.e. no errors are expected in the data format), when you have to import data processed by an operator (e.g. survey data collected on the field, recorded on paper sheets and then digitalized in a spreadsheet) you always find many errors that prevent you to import it in a (properly formatted) database table. Typical examples are notes written in a field that should be numeric (e.g. "7, but I am not sure" or ">7" related to the number of individuals observed); date and time written in a strange format or automatically converted to number by the spreadsheet tool.  
In general, there are many errors that must be fixed in these data sets (e.g. cell coloured in yellow with no explanation of what it means, numbers out of range, measures related to a survey with a date that not correspond to that of the survey, coordinates out of the study area, name of species not consistently used throughout the same data sets, valuable information not properly coded but left as notes, and many others with no limits to creativity...).  
In this case, before data are properly structured and stored in a database, the errors must be identified and corrected. This requires a reiterated exchange of information with they who collected the data and . The creation of a database from a dataset collected on the field is a very good opportunities to clean it as the database rejects inconsistency that can be easily found and fixed.  
From an operational point of view, data cleaning can be done directly on the spreadsheet, or in any other tool (e.g. R). Once you become familiar with database and SQL, you will see that a very effective and efficient way to screen data and fix problem is to import data in a database with all fields in text format (so no *a priori* checks are done on the data) and that processed using the tools offered by the database.