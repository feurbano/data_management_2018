# 3. Movement Ecology Data Management (Urbano, 6 h)
## 3.1 SETTING UP THE MOVEMENT ECOLOGY DATABASE
In this lesson, you are guided through how to set up a new database in which you will create a table to accommodate the test GPS data sets. You create a new table in a dedicated schema. This lesson describes how to upload the raw GPS data coming from five sensors deployed on roe deer in the Italian Alps into the database and how tocreate additional database users.

The datasets you will need for the following lessons can be downloaded
[here](https://github.com/feurbano/data_management_2018/tree/master/data/tracking_db.zip). Download and unzip the contents to your computer. If using Windows, unzip them to a location the `postgres` database user can access (e.g., a new folder under `C:/postgis_workshop/`).

### 3.1.1 Introduction to the goals and the data sets used for the study case on Movement Ecology Data
Once a tracking project starts and sensors are deployed on animals, data begin to arrive (usually in the form of text files containing the raw information recorded by sensors). At this point, data must be handled by researchers. The aim of this exercise is to set up an operational database where GPS data coming from roe deer monitored in the Alps can be stored, managed and analysed. The information form the sensors must be complemented with other information on the individuals, deployments and surrounding environment to have a complete picture of the animals movement.



----------------------------------<

Assuming that you have PostgreSQL installed and running on your
computer (or server), the first thing that you have to do to import
your raw sensors data into the database is to connect to the database
server and create a new database with the
command [CREATE DATABASE](http://www.postgresql.org/docs/devel/static/sql-createdatabase.html):

```sql
CREATE DATABASE gps_tracking_db
ENCODING = 'UTF8'
TEMPLATE = template0
LC_COLLATE = 'C'
LC_CTYPE = 'C';
```

You could create the database using just the first line of the
code. The other lines are added just to be sure that the database will
use UTF8 as encoding system and will not be based on any
local-specific setting regarding, e.g. alphabets, sorting, or number
formatting. This is very important when you work in an international
environment where different languages (and therefore characters) can
potentially be used. When you import textual data with different
encoding, you have to specify the original encoding otherwise special
character might be misinterpreted. Different encodings are a typical
source of error when data are moved from a system to another.

Although not compulsory, it is very important to document the objects
that you create in your database to enable other users (and probably
yourself later on) to understand its structure and content, pretty
much the same way you use to do with metadata of your
data. [COMMENT](http://www.postgresql.org/docs/devel/static/sql-comment.html)
command gives you this possibility. Comments are stored into the
database. In this case:

```sql
COMMENT ON DATABASE gps_tracking_db   
IS 'Next Generation Data Management in Movement Ecology Summer school: my database.'; 
```

By default, a database comes with the *public* schema; it is good
practice, however, to use different schemas to store user data. For
this reason you create a new
[SCHEMA](http://www.postgresql.org/docs/devel/static/sql-createschema.html)
called *main*:

```sql
CREATE SCHEMA main;
```

```sql
COMMENT ON SCHEMA main IS 'Schema that stores all the GPS tracking core data.'; 
```

Before importing the GPS data sets into the database, it is
recommended that you examine the source data (usually .dbf, .csv, or
.txt files) with a spreadsheet or a text editor to see what
information is contained. Every GPS brand/model can produce different
information, or at least organise this information in a different way,
as unfortunately no consolidated standards exist yet. The idea is to
import raw data (as they are when received from the sensors) into the
database and then process them to transform data into
information. Once you identify which attributes are stored in the
original files (for example, *GSM01438.csv*), you can create the
structure of a table with the same columns, with the correct database
[DATA TYPES](http://www.postgresql.org/docs/devel/static/datatype.html). You
can find the GPS data sets in .csv files included in the
*trackingDB\_datasets.zip* file with test data in the sub-folder
*/tracking\_db/data/sensors\_data*. The SQL code that generates the
same table structure of the source files (Vectronic GPS collars)
within the database, which is called here *main.gps\_data* (*main* is
the name of the schema where the table will be created, while
*gps\_data* is the name of the table) is

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

In a relational database, each table must have a primary key, that is
a field (or combination of fields) that uniquely identify each
record. In this case, we added a
[SERIAL](http://www.postgresql.org/docs/devel/static/sql-createsequence.html)
id field managed by the database (as a sequence of integers
automatically generated) to be sure that it is unique. We set this
field as the primary key of the table:

```sql
ALTER TABLE main.gps_data 
ADD CONSTRAINT gps_data_pkey PRIMARY KEY(gps_data_id); 
```

To keep track of database changes, it is useful to add another field
where the timestamp of the insert of each record is automatically
recorded (assigning a dynamic default value):

```sql
ALTER TABLE main.gps_data ADD COLUMN insert_timestamp timestamp with time zone; 
```

```sql
ALTER TABLE main.gps_data ALTER COLUMN insert_timestamp SET DEFAULT now(); 
```

Now we are ready to import our data sets. There are many ways to do
so. The main one is to use the
[COPY](http://www.postgresql.org/docs/devel/static/sql-copy.html)
command setting the appropriate parameters:

```sql
COPY main.gps_data( 
gps_sensors_code, line_no, utc_date, utc_time, lmt_date, lmt_time, ecef_x, ecef_y, ecef_z, latitude, longitude, height, dop, nav, validated, sats_used, ch01_sat_id, ch01_sat_cnr, ch02_sat_id, ch02_sat_cnr, ch03_sat_id, ch03_sat_cnr, ch04_sat_id, ch04_sat_cnr, ch05_sat_id, ch05_sat_cnr, ch06_sat_id, ch06_sat_cnr, ch07_sat_id, ch07_sat_cnr, ch08_sat_id, ch08_sat_cnr, ch09_sat_id, ch09_sat_cnr, ch10_sat_id, ch10_sat_cnr, ch11_sat_id, ch11_sat_cnr, ch12_sat_id, ch12_sat_cnr, main_vol, bu_vol, temp, easting, northing, remarks) 
FROM 
'C:\tracking_db\data\sensors_data\GSM01438.csv' WITH CSV HEADER DELIMITER ';'; 
```

You might have to adapt the path to the file. If you get an error
about permissions (you cannot open the source file) it means that the
file is stored in a folder where it is not accessible by the
database. In this case, move it to a folder that is accessible to all
users of your computer.

If PostgreSQL complain that date is out of range, check the standard
date format used by PostgreSQL
([datestyle](http://www.postgresql.org/docs/devel/static/runtime-config-client.html#GUC-DATESTYLE)):

```sql
SHOW datestyle; 
```

if it is not *ISO, DMY*, then you have to set the date format (for the
current session) as:

```sql
SET SESSION datestyle = "ISO, DMY"; 
```

This will change the *datastyle* for the current session only. If you
want to change this setting permanently, you have to modify the
*datestyle* option in the
[postgresql.conf](http://www.postgresql.org/docs/devel/static/config-setting.html#CONFIG-SETTING-CONFIGURATION-FILE)
file.

In the case of .dbf files, you can use a tool that comes with PgAdmin
(*Shapefile and .dbf importer*). In this case, you do not have to
create the structure of the table before importing, but you lose
control over the definition of data type (e.g. time will probably be
stored as a text value). You can also use MS Access and link both
source and destination table and run an upload query.


### Exercise

Import into the database the raw GPS data from the test sensors:

-   GSM01508
-   GSM01511
-   GSM01512

----------------------------------<



### 3.1.2 Import data into the database
### 3.1.3 Create keys, indices and domains
### 3.1.4 Managing and modelling information on animals and sensors
### 3.1.5 From data to information: associating locations to animals
### 3.1.6 Manage the location data in a spatial database
### 3.1.7 From locations to trajectories and home ranges
### 3.1.8 Integrating spatial ancillary information: land cover
### 3.1.9 Data quality: how to detect and manage outliers
### 3.1.10 Raster Data in PostGIS (demo)