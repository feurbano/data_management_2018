# 3. Movement Ecology Data Management (Urbano, 6 h)
## 3.1 SETTING UP THE MOVEMENT ECOLOGY DATABASE
In this lesson, you are guided through how to set up a new database in which you will create a table to accommodate the test GPS data sets. You create a new table in a dedicated schema. This lesson describes how to upload the raw GPS data coming from five sensors deployed on roe deer in the Italian Alps into the database and how tocreate additional database users.

The datasets you will need for the following lessons can be downloaded **[here](https://github.com/feurbano/data_management_2018/tree/master/data/tracking_db.zip)**. Download and unzip the contents to your computer. If using Windows, unzip them to a location the `postgres` database user can access (e.g., a new folder under `C:/tracking_db/`).

### 3.1.1 Introduction to the goals and the data set
Once a tracking project starts and sensors are deployed on animals, data begin to arrive (usually in the form of text files containing the raw information recorded by sensors). At this point, data must be handled by researchers. The aim of this exercise is to set up an operational database where GPS data coming from roe deer monitored in the Alps can be stored, managed and analysed. The information form the sensors must be complemented with other information on the individuals, deployments and surrounding environment to have a complete picture of the animals movement.


### 3.1.2 Create a db and import data

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

**IMPORTANT REMARK**
> *While data generated by a sensor are usually "clean" (i.e. no errors are expected in the data format), when you have to import data processed by an operator (e.g. survey data collected on the field, recorded on paper sheets and then digitalized in a spreadsheet) you always find many errors that prevent you to import it in a (properly formatted) database table. Typical examples are notes written in a field that should be numeric (e.g. "7, but I am not sure" or ">7" related to the number of individuals observed); date and time written in a stange format or automatically converted to number by the spreadsheet tool.
> In general, there are many errors that must be fixed to "clean" the data sets (e.g. cell colored in yellow with no explanation of what it means, numbers out of range, measures related to a survey with a date that not correspond to that of the survey, coordinates out of the study area, name of species not consistently used throughout the same data sets, valuable information not properly coded but left as notes, and many othes with no limits to creativity...).
> In this case, before data are prperly structured and stored in a database, the errors must be fixed. This implies a reiterated exchange of information with they who collected the data. The creation of a database from a dataset collected on the field is a very good opportunities to clean it as the database rejects inconsistency that can be easily foud and fixed.
> From an operational point of view, data cleaning can be done directly on the spreadsheet, or in any other tool (e.g. R). Once you become familiar with database and SQL, you will see that a very effective and efficient way to screen data and fix problem is to import data in a database with all fields in text format (so no *a priori* checks are done on the data) and that processed using the tools offered by the database.*

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

PgAdmin offers the possibility to import data (includeing local data to a server) with a grphical interface (right click on the table, and select *Import*). In the case of .dbf files, you can use a tool that comes with PgAdmin (*Shapefile and .dbf importer*). In this case, you do not have to create the structure of the table before importing, but you lose control over the definition of data type (e.g. time will probably be stored as a text value). You can also use MS Access and link both source and destination table and run an upload query. In general, any database front end has tools that faciitate data import form files.

#### Exercise

Import into the database the raw GPS data from the test sensors:

-   GSM01508
-   GSM01511
-   GSM01512


### 3.1.3 Create acquisition timestamps, indexes and permissions

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
* Find the length of the dataset for each animal
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

### 3.1.4 Managing and modelling information on animals and sensors
### 3.1.5 From data to information: associating locations to animals
### 3.1.6 Manage the location data in a spatial database
### 3.1.7 From locations to trajectories and home ranges
### 3.1.8 Integrating spatial ancillary information: land cover
### 3.1.9 Data quality: how to detect and manage outliers
### 3.1.10 Data export and maintenance

> Once your database is populated and used for daily work, it is a
> *really* good idea to routinely make a safe copy of your data. Since
> the RDBMS mantains data in a binary format which is not meant to be
> tampered with, we need to `dump` the database content in a format
> suitable for being later restored if it needs be. The very same dump
> could also be used for replicating the database contents on another
> server.
> 
> From pgAdmin, the operation of making a database dump is extremely
> simple: right click the database, choose `Backup...` and here you are:
> 
> There are a few output formats, apart from the default `Custom` one.
> Here I chose `Plain` because the file will then contain readable SQL
> commands, and as such it is useful to study. You should be aware that
> this format is **not suitable**for restoring from inside pgAdmin,
> since it contains commands which are not implemented in pgAdmins Query
> Tool.
> 
> The bulletproof way for importing a dump file in plain (SQL) format is
> to get back to the command line, and use our old friend, `psql`:
> 
> ```
> psql testing < testing.sql
> ```
> 
> See the docs on
> [backup and restore](http://www.postgresql.org/docs/current/static/backup-dump.html)
> for further information.


------

> <h3>Topic 3. Export your data and backup the database</h3>
> <h4>Introduction</h4>
> There are different ways to export a table or the results of a query to an external file. One is to use the command <a href="http://www.postgresql.org/docs/devel/static/sql-copy.html">COPY (TO)</a>. COPY TO (similarly to what happens with the command COPY FROM used to import data) with a file name directly write the content of a table or the result of a query to a file. The file must be accessible by the PostgreSQL user (i.e. you have to check the permission on target folder by the user ID the PostgreSQL server runs as) and the name (path) must be specified from the viewpoint of the server. This means that files can be read or write only in folders 'visible' to the database servers. If you want to remotely connect to the database and save data into your local machine, you can use the command <a href="http://www.postgresql.org/docs/devel/static/app-psql.html#APP-PSQL-META-COMMANDS-COPY">\COPY</athat performs a frontend (client) copy. \COPY is not an SQL command and must be run from a PostgreSQL interactive terminal (<a href="http://www.postgresql.org/docs/devel/static/app-psql.html">PSQL</a>). This is an operation that runs an SQL COPY command, but instead of the server reading or writing the specified file, PSQL reads or writes the file and routes the data between the server and the local file system. This means that file accessibility and privileges are those of the local user, not the server, and no SQL superuser privileges are required.
> Another possibility to export data is to use the pgAdmin interface: in the SQL console select <em>Query/Execute to file</emand the results will be saved to a local file instead of being visualized. Other database interfaces have similar tools.
> 
> A proper backup policy for a database is important to securing all your valuable data and the information that you have derived through data processing. In general it is recommended to have frequent (scheduled) backups (e.g. once a day) for schemas that change often and less frequent backups (e.g. once a week) for schemas (if any) that occupy a larger disk size and do not change often (e.g. ancillary environmental layers). PostgreSQL offers very good tools for database <a href="http://www.postgresql.org/docs/devel/static/backup.html">backup and recovery</a>. The two main tools to back up are:
> <ul>
>  	<li><a href="http://www.postgresql.org/docs/devel/static/app-pgdump.html">pg_dump.exe</a>: extracts a PostgreSQL database or part of the database into a script file or other archive file (<a href="http://www.postgresql.org/docs/devel/static/app-pgrestore.html">pg_restore.exe</ais then used to restore the database);</li>
>  	<li><a href="http://www.postgresql.org/docs/devel/static/app-pg-dumpall.html">pg_dumpall.exe</a>: extracts a PostgreSQL database cluster (i.e. all the databases created inside the same installation of PostgreSQL) into a script file (e.g. including database setting, roles).</li>
> </ul>
> These are not SQL commands but executable commands that must run from a command-line interpreter (with Windows, the default command-line interpreter is the program <em>cmd.exe</em>, also called <em>Command Prompt</em>). pgAdmin also offers a graphic interface for backing up and restoring the database. Moreover, it also important to keep a file-based copy of the original raw data files, particularly those generated by sensors.
> <h4>Example</h4>
> An example of data export for the whole <em>main.gps_data table</emis
> <pre><code class="SQL">COPY (
>   SELECT gps_data_id, gps_sensors_code, latitude, longitude, acquisition_time, insert_timestamp 
>   FROM main.gps_data) 
> TO 
>   'C:\tracking_db\test\export_test1.csv' 
>   WITH (FORMAT csv, HEADER, DELIMITER ';');
> </code></pre>
> An example of a back up (if you want to reuse this layout, you must properly st the path to your <em>pg_dump</emcommand) of the schema <em>main</emand all its content is
> <blockquote>C:\PostgreSQL\9.4\bin\pg_dump.exe --host localhost --port 5432 --username "postgres" --no-password --format plain --encoding UTF8 --verbose --file "C:\tracking_db\test\backup_db_20150706.sql" --schema "main" "gps_tracking_db"</blockquote>
> <h4>Exercise</h4>
> <ol>
>  	<li>Find and export to a .csv file the extreme coordinates (minimum and maximum longitude and latitude) and the duration of the deployment for each sensor</li>
>  	<li>Make a complete backup of the database built so far and restore it in a new database on your computer</li>
> </ol>
> 



### 3.1.11 Raster data in PostGIS (demo)