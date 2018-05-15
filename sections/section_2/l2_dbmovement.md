# <a name="c_2.1"></a>2. Movement Ecology Data Management

* **[2.1 Introduction to the goals and the datasets](#c_2.1)**
* **[2.2 Create a db and import sensor data](#c_2.2)**
* **[2.3 Create acquisition timestamps, indexes and permissions](#c_2.3)**
* **[2.4 Managing and modelling information on animals and sensors](#c_2.4)**
* **[2.5 From data to information: associating locations to animals](#c_2.5)**
* **[2.6 Manage the location data in a spatial database](#c_2.6)**
* **[2.7 From locations to trajectories and home ranges](#c_2.7)**
* **[2.8 Integrating spatial ancillary information: land cover](#c_2.8)**
* **[2.9 Data quality: how to detect and manage outliers](#c_2.9)**
* **[2.10 Data export](#c_2.10)**
* **[2.11 Database maintenance](#c_2.11)**
* **[2.12 Recap exercises](#c_2.12)**
* **[2.13 Raster Data in PostGIS (demo)](https://github.com/feurbano/data_management_2018/blob/master/sections/section2/l2.13_raster.md)**
* **[2.14 Functions and triggers (supplementary material](https://github.com/feurbano/data_management_2018/blob/master/sections/section2/l2.14_supplementary.md)**
  * [Timestamping changes in the database using triggers](#c_2.14.1)
  * [Automation of the GPS data association with animal](#c_2.14.2)
  * [Consistency checks on the deployments information](#c_2.14.3)
  * [Synchronization of *gps\_sensors\_animals* and *gps\_data\_animals*](#c_2.14.4)
  * [Automating the creation of points from GPS coordinates](#c_2.14.5)
  * [UTM zone of a given point in geographic coordinates](#c_2.14.6)

## <a name="c_2.1"></a>2.1 Introduction to the goals and the data set
Once a tracking project starts and sensors are deployed on animals, data begin to arrive (usually in the form of text files containing the raw information recorded by sensors). At this point, data must be handled by researchers. The aim of this exercise is to set up an operational database where GPS data coming from roe deer monitored in the Alps can be stored, managed and analysed. The information form the sensors must be complemented with other information on the individuals, deployments and surrounding environment to have a complete picture of the animals movement.

In this lesson, you are guided through how to set up a new database in which you will create a table to accommodate the test GPS data sets. You create a new table in a dedicated schema. This lesson describes how to upload the raw GPS data coming from five sensors deployed on roe deer in the Italian Alps into the database and how tocreate additional database users.

The datasets you will need for the following lessons can be downloaded **[here](https://github.com/feurbano/data_management_2018/tree/master/data/tracking_db.zip)**. Download and unzip the contents to your computer. If using Windows, unzip them to a location the `postgres` database user can access (e.g., a new folder under `C:/tracking_db/`).

## <a name="c_2.2"></a>2.2 Create a db and import sensor data

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

1. Import into the database the raw GPS data from the test sensors:
  -   GSM01508
  -   GSM01511
  -   GSM01512

## <a name="c_2.3"></a>2.3 Create acquisition timestamps, indexes and permissions

### Acquisition timestamps
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
1. Find the temporal duration of the dataset for each animal
*Hint: get the minimum and maximum value of acquisition time for each animal and then make the difference to have the interval*

2. Retrieve data from the collar **GSM01512** during the month of May (whatever the year), and order them by their acquisition time
*Hint: use extract (month ...) to set the criteria on month*

### Indexes

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

### Permissions

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

## <a name="c_2.4"></a>2.4 Managing and modelling information on animals and sensors
GPS positions are used to describe animal movements and to derive a large set of information, for example about animals' behavior, social interactions, and environmental preferences. GPS data are related to (and must be integrated with) many other sources of information that together can be used to describe the complexity of movement ecology. In a database framework, this can only be achieved through proper database data modelling, which depends on a clear definition of the biological context of a study. Particularly, data modelling becomes a key step when database systems manage a large set of connected data sets that grow in size and complexity: it permits easy updates and modification and adaptation of the database structure to accommodate the changing goals, constraints, and spatial scales of studies. 
In this lesson, you will extend your database with two new tables to integrate ancillary information useful to interpreting GPS data: one for GPS sensors, and one for animals. Additionally, you will add "lookup tables" which translate certain codes into descriptions, and enforce which codes can be used for species and age classes in the database.

### The world in a database: database data models
A data model describes what types of data are stored and how they are organized. It can be seen as the conceptual representation of the real world in the database structures that include data objects (i.e. tables) and their mutual relationships. In particular, data modelling becomes a key step when database systems grow in size and complexity, and user requirements become more sophisticated: it permits easy updates and modification and adaptation of the database structure to accommodate the changing goals, constraints, and spatial scales of studies and the evolution of wildlife tracking systems. Without a rigorous data modelling approach, an information system might lose the flexibility to manage data efficiently in the long term, reducing its utility to a simple storage device for raw data, and thus failing to address many of the necessary requirements.

To model data properly, you have to clearly state the biological context of your study. A logical way to proceed is to define (**a**) very basic questions on the sample unit, i.e. individual animals and (**b**) basic questions about data collection. **a**) Typically, individuals are the sampling units of an ecological study based on wildlife tracking. Therefore, the first question to be asked while modelling the data is: *What basic biological information is needed to characterise individuals as part of a sample?* Species, sex and age (or age class) at capture are the main factors which are relevant in all studies. Age classes typically depend on the species. Moreover, age class is not constant for all the GPS positions. The correct age class at any given moment can be derived from the age class at capture and then defining rules that specify when the individual change from a class to another (for roe deer, you might assume that at 1st April of every year each individual that was a fawn becomes a yearling, and each yearling becomes an adult). Other information used to characterise individuals could be specific to a study, for example in a study on spatial behaviour of translocated animals, 'resident' or 'translocated' is an essential piece of information linked to individual animals. All these elements should be described in specific tables. **b**) A single individual becomes a 'studied unit' when it is fitted with a sensor, in this case to collect position information. First of all, GPS sensors should be described by a dedicated table containing the technical characteristics of each device (e.g. vendor, model). Capture time, or 'device-fitting' time, together with the time and a description of the end of the deployment (e.g. drop-off of the tag, death of the animal), are also essential to the model. The link between the sensors and the animals should be described in a table that states unequivocally when the data collected from a sensor 'become' (and cease to be) bio-logged data, i.e. the period during which they refer to an individual's behaviour. The start of the deployment usually coincides with the moment of capture, but it is not the same thing. Indeed, moment of capture can be the 'end' of one relationship between a sensor and an animal (i.e. when a device is taken off an animal) and at the same time the 'beginning' of another (i.e. another device is fitted instead).

Thanks to the tables 'animals', 'sensors' and 'sensors to animals', and the relationships built among them, GPS data can be linked unequivocally to individuals, i.e. the sampling units.

Some information related to animals can change over time. Therefore, they must be marked with the reference time that they refer to. Examples of typical parameters assessed at capture are age and positivity of association to a disease. Translocation may also coincide with the capture/release time. If this information changes over time according to well-defined rules (e.g. transition from age classes), their value can be dynamically calculated in the database at different moments in time (e.g. using database functions). In one of the next lessons, you will see an example of a function to calculate age class from the information on the age class at capture and the acquisition time of GPS positions for roe deer. The basic structure based on the elements *animals*, *sensors*, *sensors to animals*, and, of course, *position data*, can be extended to take into account the specific goals of each project, the complexity of the real-world problems faced, the technical environment, and the available data. Examples of data that can be integrated are capture methodology, handling procedure, use of tranquilizers and so forth, that should be described in a 'captures' table linked to the specific individual (in the table 'animals'). Finally, data referring to individuals may come from several sources, e.g. several sensors or visual observations. In all these cases, the link between data and sample units (individuals) should also be clearly stated by appropriate relationships. In complex projects, especially those involving field data collection in parallel with remote tracking, and involve multiple species/sensor types, databases can quickly get complex and grow up to hundreds of tables.

The design of the database data model is an exercise that must be performed at the very initial stage of the creation of a database. Once the objects (and objectives) of a study are identified and described (see above), some tools exist to graphically translates the conceptual model into connected tables, each of them representing a specific entity of the world. This process is not trivial and force biologists to "formalize" their goals, data and scientific approach (which also helps to organize the whole data collection in a systematic and consistent way). For example, at the beginning of a tracking study, it is easy to assume that a tag (i.e. a collar) can be identified with the animal where it is deployed, creating a single table. Later on, it happens very often that the same collar is reused on other animals, thus making a data model based on a single animal-collar table unsuitable. Changing a database on and advanced stage of development is very complicate and requires by far more times than a carefully planned phase of data modelling at the start of the project.

A very popular graphical tool to represent a data model is the **[Entity-Relationship Diagram (ERD)](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model)** that helps to show the relationship between elements, concepts or events of a system and that can be used as the foundation for a relational database.

In the figure below, it is illustrated the schema of the database structure created at the end of this and the next lessons.

<p align="center">
<img src="https://github.com/feurbano/data_management_2018/blob/master/sections/section_2/images/schema-db.png" Height="450"/>
</p>

### Extend the database: data on sensors

At the moment, there is a single table in the your test database and it represents raw data from GPS sensors. This data alone gives very little information on what it represents. To take full advantage of these positional data sets, you have to join locations with other kind of information that help to transform the raw number into the description of real-life objects (in this case, moving animals). When this contextual information is very limited, you can simply add some metadata, but in case it is more structured and complex, you have to properly integrate it into the database as tables. Now you can start to extend the database with new tables to represent sensors and animals. This process will continue throughout all the following lessons in order to include a wide range of information that are relevant for wildlife tracking.

In the sub-folder */tracking\_db/data/animals* and */tracking\_db/data/sensors* of the test data set you will find two files:*animals.csv* and *gps\_sensors.csv*. Let's start with data on GPS sensors. Once you have explored its content, first you have to create a table in the database with the same attributes as the .csv file, and then import the data into it. Here is the code of the table structure:

```sql
CREATE TABLE main.gps_sensors(
  gps_sensors_id integer,
  gps_sensors_code character varying NOT NULL,
  purchase_date date,
  frequency double precision,
  vendor character varying,
  model character varying,
  sim character varying,
CONSTRAINT gps_sensors_pkey
PRIMARY KEY (gps_sensors_id),
CONSTRAINT gps_sensor_code_unique
UNIQUE (gps_sensors_code)
);
```

```sql
COMMENT ON TABLE main.gps_sensors
IS 'GPS sensors catalog.';
```

The field *gps\_sensors\_id* is an integer and is used as the primary key. You could also use *gps\_sensors\_code* as primary key, but in many practical situations it is handy to use an integer field. In some cases, a good recommendation is to use a *serial number* as primary key to let the database generate a unique code (integer) every time that a new record is inserted. In this exercise, we use an integer data type because the values of the *gps\_sensors\_id* field are pre-defined in order to be correctly referenced in the exercises of the next lessons. A **[UNIQUE](http://www.postgresql.org/docs/9.3/static/ddl-constraints.html)** constraint is created on the field *gps\_sensors\_code* to be sure that the same sensor is not imported more than once. You add a field to keep track of the timestamp of record insertion, which can be very useful to monitor database activities:

```sql
ALTER TABLE main.gps_sensors 
  ADD COLUMN insert_timestamp timestamp with time zone DEFAULT now();
```

Now you can import data using the COPY command (or through the pgAdmin GUI, if you prefer):

```sql
COPY main.gps_sensors(
  gps_sensors_id, gps_sensors_code, purchase_date, frequency, vendor, model, sim)
FROM 
  'C:\tracking_db\data\sensors\gps_sensors.csv' 
  WITH (FORMAT csv, DELIMITER ';');
```

At this stage, you have defined the list of GPS sensors that exist in your database. To be sure that you will never have GPS data that come from a GPS sensor that does not exist in the database, you apply a **[foreign key](http://www.postgresql.org/docs/9.3/static/ddl-constraints.html)** between *main.gps\_data* and *main.gps\_sensors*. Foreign keys physically translate the concept of relations among tables.

```sql
ALTER TABLE main.gps_data
  ADD CONSTRAINT gps_data_gps_sensors_fkey 
  FOREIGN KEY (gps_sensors_code)
  REFERENCES main.gps_sensors (gps_sensors_code) 
  MATCH SIMPLE ON UPDATE NO ACTION ON DELETE NO ACTION;
```

This setting says that in order to delete a record in *main.gps\_sensors*, you first have to delete all the associated records in *main.gps\_data*. From now on, before importing GPS data from a sensor, you have first to create the sensor's record in the *main.gps\_sensors table*. You can add other kinds of constraints to control the consistency of your database. As an example, here you check that the date of purchase is after *2000-01-01*. If this condition is not met, the database will refuse to insert (or modify) the record and will return an error message.

```sql
ALTER TABLE main.gps_sensors
  ADD CONSTRAINT purchase_date_check 
  CHECK (purchase_date > '2000-01-01'::date);
```

#### Exercise

1.  Write a query to view all the data stored in *main.gps\_data*, along with the date when the sensor was purchased and the vendor name.


### Extend the database: data on animals
The coordinates provided by GPS sensors describe in detail the spatial patterns of animal movements, which provide valuable information to biologists and wildlife managers. On the other hand, coordinates alone, with no information of what they represent i.e. (the object that is moving and its characteristics), can just partially address the main ecological questions that they can potential answer. In this and in the next lessons, we will try to build up a (database) framework to move from coordinates to more complex object: individual animals, with their characteristics and interactions, moving in their environment. The first step is to describe the individuals that are monitored. In this exercise we create a table to store basic information such as age, sex, species, and the name that researchers use to identify that individual (if any).

Now you repeat the same process for data on animals. Analyzing the animals' source file (*animals.csv*), you can derive the fields of the new *main.animals* table:

```sql
CREATE TABLE main.animals(
  animals_id integer,
  animals_code character varying(20) NOT NULL,
  name character varying(40),
  sex character(1),
  age_class_code integer,
  species_code integer,
  note character varying,
  CONSTRAINT animals_pkey PRIMARY KEY (animals_id)
);
```

```sql
COMMENT ON TABLE main.animals
IS 'Animals catalog with the main information on individuals.';
```

As for *main.gps\_sensors*, in your operational database you could have used the serial data type for the *animals\_id* field. Age class (at capture) and species are attributes that can only have defined values. To enforce consistency in the database, in these cases you can use look up tables. Look up tables are tables that store the list and the description of all possible values referenced by specific fields in different tables and constitute the definition of the valid domain. They are very common because they can help to simplify a database structure and add flexibility as compared to *constraints* defined on specific fields. It is recommended to keep look up tables in a separated schema to give the database a more readable and clear data structure. Therefore, you create a *lu\_tables* schema:

```sql
CREATE SCHEMA lu_tables;
GRANT USAGE ON SCHEMA lu_tables TO basic_user;
```

```sql
COMMENT ON SCHEMA lu_tables
IS 'Schema that stores look up tables.';
```
 You set as default that the user *basic\_user* will be able to run SELECT queries on all the tables that will be created into this schema:

```sql
ALTER DEFAULT PRIVILEGES 
  IN SCHEMA lu_tables 
  GRANT SELECT ON TABLES 
  TO basic_user;
```

Now you create a look up table for species:

```sql
CREATE TABLE lu_tables.lu_species(
  species_code integer,
  species_description character varying,
  CONSTRAINT lu_species_pkey 
  PRIMARY KEY (species_code)
);
```

```sql
COMMENT ON TABLE lu_tables.lu_species
IS 'Look up table for species.';
```

You populate it with some values (just roe deer code will be used in our test data set):

```sql
INSERT INTO lu_tables.lu_species 
  VALUES (1, 'roe deer');

INSERT INTO lu_tables.lu_species 
  VALUES (2, 'rein deer');

INSERT INTO lu_tables.lu_species 
  VALUES (3, 'moose');
```

You can do the same for age classes (note an alternative way to specify the primary key in this table definition):

```sql
CREATE TABLE lu_tables.lu_age_class(
  age_class_code integer primary key, 
  age_class_description character varying
);
```

```sql
COMMENT ON TABLE lu_tables.lu_age_class
IS 'Look up table for age classes.';
```

You populate it with some values (these categories are based on roe deer, other species would have their own approach):

```sql
INSERT INTO lu_tables.lu_age_class 
  VALUES (1, 'fawn'), (2, 'yearling'), (3, 'adult');
```

At this stage you can create the foreign keys between the *main.animals* table and the two look up tables. This will prevent to have animals with species and age classes that are not listed in the look up tables. This is an important tool to ensure data integrity.

```sql
ALTER TABLE main.animals
  ADD CONSTRAINT animals_lu_species 
  FOREIGN KEY (species_code)
  REFERENCES lu_tables.lu_species (species_code) 
  MATCH SIMPLE ON UPDATE NO ACTION ON DELETE NO ACTION;
```

```sql
ALTER TABLE main.animals
  ADD CONSTRAINT animals_lu_age_class 
  FOREIGN KEY (age_class_code)
  REFERENCES lu_tables.lu_age_class (age_class_code) 
  MATCH SIMPLE ON UPDATE NO ACTION ON DELETE NO ACTION;
```

For sex class of deer, you do not expect to have more than the two possible values: female and male (stored in the database as 'f' and 'm' to simplify data input). In this case, instead of a look up table you can set a check on the field:

```sql
ALTER TABLE main.animals
  ADD CONSTRAINT sex_check 
  CHECK (sex = 'm' OR sex = 'f');
```

Whether it is better to use a look up table or a check must be evaluated case by case, mainly according to the number of admitted values and the possibility that you will want to add new values in the future. You should also add a field to keep track of the timestamp of record insertion in order to be able to keep track of what happened in the database. More complex approach are possible to monitor the database activity, from the creation of a log file that store all the operations performed to proper versioned database, each of them with pro and cons.

```sql
ALTER TABLE main.animals 
  ADD COLUMN insert_timestamp timestamp with time zone DEFAULT now();
```

As a last step, you import the values from the file:

```sql
COPY main.animals(
  animals_id,animals_code, name, sex, age_class_code, species_code)
FROM 
  'C:\tracking_db\data\animals\animals.csv' 
  WITH (FORMAT csv, DELIMITER ';');
```

To test the result, you can retrieve the animals' data with the extended age class description:

```sql
SELECT
  animals.animals_id AS id, 
  animals.animals_code AS code, 
  animals.name, 
  lu_age_class.age_class_description AS age_class
FROM 
  lu_tables.lu_age_class, 
  main.animals
WHERE 
  lu_age_class.age_class_code = animals.age_class_code 
```

#### Exercise

1.  Query the table *main.animals* including the information on the sex and the species (with the name of the species stored in the look up table)
2.  Modify the field *vendor* in the table *main.gps\_sensors* to limit the possible values to: 'Vectronic Aerospace GmbH', 'Sirtrack' and 'Lotek', then try to insert a vendor not in the list.
3.  How would you extend the database to include information on mortality (date and type of death)
4.  Design a general schema of a possible database extension (i.e. table(s) and their links) to include information on capture and particularly on:
    -   date and time of the capture (note that the same animal can be captured more than once)
    -   location of capture
    -   capture method
    -   person who captured the animal
    -   if the animal was collared, in this case include the code of the collar
    -   the name/id of the animal
    -   body temperature (note that temperature can be measured more than once at the different stages of the capture).




## <a name="c_2.5"></a>2.5 From data to information: associating locations to animals
When position data are received from GPS sensors, they are not explicitly associated with any animal. Linking GPS data to animals is a key step in the data management process. This can be achieved using the information on the deployments of GPS sensors on animals (when sensors started and ceased to be deployed on the animals). In the case of a continuous data flow, the transformation of GPS positions into animal locations must be automated in order to have GPS data imported and processed in real-time. In this lesson, you extend the database with two new tables, *gps\_sensors\_animals* and *gps\_data\_animals*. As additional material, a set of dedicated database triggers and functions is presented that add tools to automatically manage the association of GPS positions with animals.


### Storing information on GPS sensor deployments on animals
To associate a positions to animals, you need the information on deployments, i.e. the time interval when a defined animal is wearing a specific tag. This information is also needed to exclude the positions recorded when a sensor was not deployed on any animal. The key point is to integrate into the database the information on the deployment of sensors on animals in a dedicated table. The design of the table structure must take into consideration that each animal can be monitored with multiple sensors (most likely at different times) and each sensor can be reused on multiple animals (no more than one at a time). This corresponds to a many-to-many relationship between animals and GPS sensors, where the main attribute is the time range: start and end of deployment (when the sensor is still deployed on the animal, the end of deployment can be set to null). Making reference to the case of GPS (but with a general validity), this information can be stored in a *gps\_sensors\_animals* table where the ID of the sensor, the ID of the animal, and the start and end timestamps of deployment are included. A possible solution to store the records that are associated with animals is to create a new table, which could be called *gps\_data\_animals*, where a list of derived fields can be eventually added to the basic animals ID, GPS sensors ID, acquisition time, and coordinates. This new table duplicates part of the information stored in the original *gps\_data table*, and the two tables must be kept synchronized. On the other hand, there are advantages of this database structure over alternative approaches with a single table (*gps\_data*) where all the original data from GPS sensors (including GPS positions not associated to animals) are also related to other information (e.g. the animal ID, environmental attributes, movement parameters):

-   *gps\_data* cannot be easily synchronized with the data source if too many additional fields (i.e. calculated after data are imported into the database) are present in the table;
-   if sensors from different vendors or different models of the same vendor are used, you might have file formats with a different set of fields: in this case it is complicated to merge all the information from each source in a single *gps\_data*table;
-   in a table containing just those GPS positions associated with animals, performance improves because of the reduced number of records and fields (the fields not relevant for analysis, e.g. the list of satellites, can be kept only in the *gps\_data* table);
-   with the additional *gps\_data\_animals* table, it is easier and more efficient to manage a system of tags to mark potential wrong locations and to share and disseminate the relevant information (you would lose the information on outliers if *gps\_data* is synchronized with the original data set, i.e. the text file from the sensors). In this course, we use the table *gps\_data* as an exact copy of raw data as they come from GPS sensors, while *gps\_data\_animals* is used to store and process the information that is used to monitor and study animals' movements. In a way,*gps\_data* is a 'system' table used as an intermediate step for the data import process. This approach is an example of possible database structure, but there is no best solution that fits all cases and a database for wildlife tracking data must be designed according to the specific data and requirements of each project.

The first step to associating GPS positions with animals is to create a table to accommodate information on the deployment of GPS sensors on animals:

```sql
CREATE TABLE main.gps_sensors_animals(
  gps_sensors_animals_id serial NOT NULL, 
  animals_id integer NOT NULL, 
  gps_sensors_id integer NOT NULL,
  start_time timestamp with time zone NOT NULL, 
  end_time timestamp with time zone,
  notes character varying, 
  insert_timestamp timestamp with time zone DEFAULT now(),
  CONSTRAINT gps_sensors_animals_pkey 
    PRIMARY KEY (gps_sensors_animals_id ),
  CONSTRAINT gps_sensors_animals_animals_id_fkey 
    FOREIGN KEY (animals_id)
    REFERENCES main.animals (animals_id) 
    MATCH SIMPLE ON UPDATE NO ACTION ON DELETE CASCADE,
  CONSTRAINT gps_sensors_animals_gps_sensors_id_fkey 
    FOREIGN KEY (gps_sensors_id)
    REFERENCES main.gps_sensors (gps_sensors_id) 
    MATCH SIMPLE ON UPDATE NO ACTION ON DELETE CASCADE
);
```

```sql
COMMENT ON TABLE main.gps_sensors_animals
IS 'Table that stores information of deployments of sensors on animals.';
```

 Now your table is ready to be populated. The general way of populating this kind of table is the manual entry of the information. In our case, you can use the test data set stored in the .csv file included in the test data set *\\tracking\_db\\data\\sensors\_animals\\gps\_sensors\_animals.csv*:

```sql
COPY main.gps_sensors_animals(
  animals_id, gps_sensors_id, start_time, end_time, notes)
FROM 
  'C:\tracking_db\data\sensors_animals\gps_sensors_animals.csv' 
  WITH (FORMAT csv, DELIMITER ';');
```

Once the values in this table are updated, you can use an SQL statement to obtain the id of the animal related to each GPS position. Here an example of a query to retrieve the codes of the animal and GPS sensor, the acquisition time and the coordinates (to make the query easier to read, aliases are used for the name of the tables):

```sql
SELECT 
  deployment.gps_sensors_id AS sensor, 
  deployment.animals_id AS animal,
  data.acquisition_time, 
  data.longitude::numeric(7,5) AS long, 
  data.latitude::numeric(7,5) AS lat
FROM 
  main.gps_sensors_animals AS deployment,
  main.gps_data AS data,
  main.gps_sensors AS gps
WHERE 
  data.gps_sensors_code = gps.gps_sensors_code AND
  gps.gps_sensors_id = deployment.gps_sensors_id AND
  (
    (data.acquisition_time >= deployment.start_time AND 
     data.acquisition_time <= deployment.end_time)
    OR 
    (data.acquisition_time >= deployment.start_time AND 
     deployment.end_time IS NULL)
  )
ORDER BY 
  animals_id, acquisition_time
LIMIT 5;
```

In the query, three tables are involved: *main.gps\_sensors\_animals*, *main.gps\_data*, and *main.gps\_sensors*. This is because in the *main.gps\_data*, where raw data from the sensors are stored, the *gps\_sensors\_id* is not present, thus the table *main.gps\_sensors* is necessary to convert the *gps\_sensors\_code* into the corresponding *gps\_sensors\_id*. You can see that in the WHERE part of the statement two cases are considered: when the acquisition time is after the start and before the end of the deployment, and when the acquisition time is after the start of the deployment and the end is NULL (which means that the sensor is still deployed on the animal). 

While in general in animal tracking most of the data come from sensors, still there are data generated by humans that in many cases contain errors. If you did not collar the animals by yourself, the information on deployment will be probably provided by the operators who collared the animals. You have to carefully check the logic consistency of the data. Even if the set of information is limited (animal, sensor, start and end time of deployment, in some cases the reason for the end of the deployment), you can expect problems. Clearly, not all the errors can be detected, but there are some checks that help to identify the main ones. Typical issues are:

* the time of deployment is not provided (but it is very important!), only the date
* the time zone of the start/end time is not provided
* the period of deployment is suspicious (very long or very short)
* the period of deployment do not match with the timestamp of the locations in the sensor data file (it is longer, or much shorter)
* once visualized the first locations show an unusual spatial pattern (e.g. along the highway between the research centre and the study area)

All these things must be verified with they who provided the data.
Another typical problem is the name of the animal. In some research groups, it happens to assign the code of the GPS sensor as name to animals, but when the same animal is monitored with another collar or the same collar is deployed on another animal, it is no more possible to related the data to the correct individual. Unusual patterns in the name of the animals is also a point to control.

Data coming from field surveys with hundred of field compiled by humans in a spreadsheet, can be a nightmare to clean before they can be imported into a database. This is even more complicate when data comes from historical surveys. In general, there are many errors that must be fixed in these data sets (e.g. notes written in a field that should be numeric, cell coloured in yellow with no explanation of what it means, numbers out of range, measures related to a survey with a date that not correspond to that of the survey, coordinates out of the study area, name of species not consistently used throughout the same data sets, valuable information not properly coded but left as notes, and many others with no limits to creativity...). In this case, before data are properly structured and stored in a database, the errors must be identified and corrected. This requires a reiterated exchange of information with they who collected the data and . The creation of a database from a dataset collected on the field is a very good opportunities to clean it as the database rejects inconsistency that can be easily found and fixed. From an operational point of view, data cleaning can be done directly on the spreadsheet, or in any other tool (e.g. R). Once you become familiar with database and SQL, you will see that a very effective and efficient way to screen data and fix problem is to import data in a database with all fields in text format (so no *a priori* checks are done on the data) and that processed using the tools offered by the database.

##### Exercise

1.  Calculate how many locations for each sensor are not related to any animal (i.e. are not inside any deployment period)
2.  Create a constraint on *main.gps\_sensors\_animals* to avoid the case where *start\_time* &gt; *end\_time*

### From GPS position to animal locations

Once the information on GPS sensors deployment is included into the database, it is possible to associate locations to animals. It is convenient to store this information in a permanent table.

A new table (*main.gps\_data\_animals*) can be used to store GPS data with the code of the animal were the related sensor is deployed. This table will be the main reference for data analysis, visualization, and dissemination. In the figure below it is illustrated the structure of data flow that populates the *gps\_data\_animals* table.

<p align="center">
<img src="https://github.com/feurbano/data_management_2018/blob/master/sections/section_2/images/deployment-schema.png" Height="300"/>

Here is the SQL code that generates the *main.gps\_data\_animals* table:

```sql
CREATE TABLE main.gps_data_animals(
  gps_data_animals_id serial NOT NULL, 
  gps_sensors_id integer, 
  animals_id integer,
  acquisition_time timestamp with time zone, 
  longitude double precision,
  latitude double precision,
  insert_timestamp timestamp with time zone DEFAULT now(), 
  CONSTRAINT gps_data_animals_pkey 
    PRIMARY KEY (gps_data_animals_id),
  CONSTRAINT gps_data_animals_animals_fkey 
    FOREIGN KEY (animals_id)
    REFERENCES main.animals (animals_id) 
    MATCH SIMPLE ON UPDATE NO ACTION ON DELETE NO ACTION,
  CONSTRAINT gps_data_animals_gps_sensors 
    FOREIGN KEY (gps_sensors_id)
    REFERENCES main.gps_sensors (gps_sensors_id) 
    MATCH SIMPLE ON UPDATE NO ACTION ON DELETE NO ACTION
);
```

```sql
COMMENT ON TABLE main.gps_data_animals 
IS 'GPS sensors data associated to animals wearing the sensor.';
```

```sql
CREATE INDEX gps_data_animals_acquisition_time_index
  ON main.gps_data_animals
  USING BTREE (acquisition_time);
```

```sql
CREATE INDEX gps_data_animals_animals_id_index
  ON main.gps_data_animals
  USING BTREE (animals_id);
```

The foreign keys (on *animals\_id* and *sensor\_id*) are created as well as indexes on the fields *animals\_id* and *acquisition\_time* (that are those most probably involved in queries on this table). 

At this point, you can feed the table with the data stored in the table *gps\_data* and use *gps\_sensors\_animals* to derive the id of the animals (checking if the timestamp of the location falls inside the deployment interval of the sensor on an animal):

```sql
INSERT INTO main.gps_data_animals (
  animals_id, gps_sensors_id, acquisition_time, longitude, latitude) 
SELECT 
  gps_sensors_animals.animals_id,
  gps_sensors_animals.gps_sensors_id,
  gps_data.acquisition_time, gps_data.longitude,
  gps_data.latitude
FROM 
  main.gps_sensors_animals, main.gps_data, main.gps_sensors
WHERE 
  gps_data.gps_sensors_code = gps_sensors.gps_sensors_code AND
  gps_sensors.gps_sensors_id = gps_sensors_animals.gps_sensors_id AND
  (
    (gps_data.acquisition_time>=gps_sensors_animals.start_time AND 
     gps_data.acquisition_time<=gps_sensors_animals.end_time)
    OR 
    (gps_data.acquisition_time>=gps_sensors_animals.start_time AND 
     gps_sensors_animals.end_time IS NULL)
  );
```

Another possibility is to simultaneously create and populate the table *main.gps\_data\_animals* by using '*CREATE TABLE main.gps\_data\_animals AS*' instead of '*INSERT INTO main.gps\_data\_animals*' in the previous query and then adding the primary and foreign keys and indexes to the table.

##### Exercise

1.  What is the proportion of records with non-null coordinates for each animal?
2.  Calculate the average longitude and latitude of all females.
3.  Calculate the dates and location (latitude and longitude) of the first and last collected location per animal.
4.  Create a view that returns, for each animal, the number of non null locations, the number of null locations, and the start and end date of the deployment.

## <a name="c_2.6"></a>2.6 Manage the location data in a spatial database
A wildlife tracking data management system must include the capability to explicitly deal with the spatial component of movement data. GPS tracking data are sets of spatio-temporal objects (locations) and the spatial component must be properly handled. You will now extend the database adding spatial functionalities through the PostgreSQL spatial extension called PostGIS. PostGIS introduces the spatial data types (both vector and raster) and a large set of SQL spatial functions and tools, including spatial indexes. This possibility essentially allows you to build a GIS using the existing capabilities of relational databases. In this lesson, you will implement a system that automatically transforms the GPS coordinates generated by GPS sensors from a pair of numbers into spatial objects.

At the moment, your data are stored in the database and the GPS positions are linked to individuals. While time is properly managed, coordinates are still just two decimal numbers (longitude and latitude) and not spatial objects. It is therefore not possible to find the distance between two points, or the length of a trajectory, or the speed and angle of the step between two locations. In this section, you will learn how to add a spatial extension to your database and transform the coordinates into a spatial element (i.e. a point). 

The first step to do in order to spatially enable your database is to load the PostGIS extension, which can easily done with the following SQL command (many other extensions exist for PostgreSQL):

```sql
CREATE EXTENSION postgis;
```

Now you can use and exploit all the features offered by PostGIS in your database. The vector objects (points, lines, and polygons) are stored in a specific field of your tables as spatial data types. This field contains the structured list of vertexes, i.e. coordinates of the spatial object, and also includes its reference system. The PostGIS spatial (vectors) data types are not topological, although, if needed, PostGIS has a dedicated
**[topological extension](http://postgis.refractions.net/docs/Topology.html)**.

With PostGIS activated, you can create a field with geometry data type in your table (2D point feature with longitude/latitude WGS84 as reference system):

```sql
ALTER TABLE main.gps_data_animals 
  ADD COLUMN geom geometry(Point,4326);
```

You can create a spatial index:

```sql
CREATE INDEX gps_data_animals_geom_gist
  ON main.gps_data_animals
  USING gist (geom);
```

You can now populate it (excluding points that have no latitude/longitude):

```sql
UPDATE 
  main.gps_data_animals
SET 
  geom = ST_SetSRID(ST_MakePoint(longitude, latitude),4326)
WHERE 
  latitude IS NOT NULL AND longitude IS NOT NULL;
```

At this point, it is important to visualize the spatial content of your tables. PostgreSQL/PostGIS offers no tool for spatial data visualization, but this can be done by a number of client applications, in particular GIS desktop software like **[ESRI ArcGIS 10.x](http://www.esri.com/software/arcgis)** or **[QGIS](http://www.qgis.org/)**. 

##### Exercise

1.  Find the distance of all locations of animal 2 to the point
    (11.0620855, 45.9878812).

## <a name="c_2.7"></a>2.7 From locations to trajectories and home ranges

GPS coordinates correspond to the raw information recorded by the sensor deployed on the animal. These can be represented in different ways: points, trajectory, home range etc. All these spatial representations are derived from the GPS coordinates: they are different ways to look at the same data. It is convenient that the way you deal with your coordinates does not modify the data itself. You can generate all these spatial-derived objects (points, trajectories, polygons) on fly using views. This has the advantage that all the representations are always synchronized with the original data sets and dynamically updated. In this section you will learn how you can do it.

**[Views](http://www.postgresql.org/docs/devel/static/sql-createview.html)** are queries permanently stored in the database. For users (and client applications), they work like normal tables, but their data are calculated at query time and not physically stored. Changing the data in a table alters the data shown in subsequent invocations of related views. Views are useful because they can represent a subset of the data contained in a table; can join and simplify multiple tables into a single virtual table; take very little space to store, as the database contains only the definition of a view (i.e. the SQL query), not a copy of all the data it presents; and provide extra security, limiting the degree of exposure of tables to the outer world. On the other hand, a view might take some time to return its data content. For complex computations that are often used, it is more convenient to store the information in a permanent table. 

You can create views where derived information is (virtually) stored. First, create a new schema where all the analysis can be accommodated and kept separated from the basic data:

```sql
CREATE SCHEMA analysis
  AUTHORIZATION postgres;
  GRANT USAGE ON SCHEMA analysis TO basic_user;
```

```sql
COMMENT ON SCHEMA analysis 
IS 'Schema that stores key layers for analysis.';
```

```sql
ALTER DEFAULT PRIVILEGES 
  IN SCHEMA analysis 
  GRANT SELECT ON TABLES 
  TO basic_user;
```

You can see below an example of a view in which just (spatially valid) positions of a single animal are included, created by joining the information with the animal and look-up tables.

```sql
CREATE VIEW analysis.view_gps_locations AS 
  SELECT 
    gps_data_animals.gps_data_animals_id, 
    gps_data_animals.animals_id,
    animals.name,
    gps_data_animals.acquisition_time at time zone 'UTC' AS time_utc, 
    animals.sex, 
    lu_age_class.age_class_description, 
    lu_species.species_description,
    gps_data_animals.geom
  FROM 
    main.gps_data_animals, 
    main.animals, 
    lu_tables.lu_age_class, 
    lu_tables.lu_species
  WHERE 
    gps_data_animals.animals_id = animals.animals_id AND
    animals.age_class_code = lu_age_class.age_class_code AND
    animals.species_code = lu_species.species_code AND 
    geom IS NOT NULL;
```

```sql
COMMENT ON VIEW analysis.view_gps_locations
IS 'GPS locations.';
```

Although the best way to visualize this view is in a GIS environment (in QGIS you might need to explicitly define the unique identifier of the view, i.e. *gps\_data\_animals\_id*), you can query its non-spatial content with

```sql
SELECT 
  gps_data_animals_id AS id, 
  name AS animal,
  time_utc, 
  sex, 
  age_class_description AS age, 
  species_description AS species
FROM 
  analysis.view_gps_locations
LIMIT 5;
```

Now you create view with a different representation of your data sets. In this case you derive a trajectory from GPS points. You have to order locations per animal and per acquisition time; then you can group them (animal by animal) in a trajectory (stored as a view):

```sql
CREATE VIEW analysis.view_trajectories AS 
  SELECT 
    animals_id, 
    ST_MakeLine(geom)::geometry(LineString,4326) AS geom 
  FROM 
    (SELECT animals_id, geom, acquisition_time 
    FROM main.gps_data_animals 
    WHERE geom IS NOT NULL 
    ORDER BY 
    animals_id, acquisition_time) AS sel_subquery 
  GROUP BY 
    animals_id;
```

```sql
COMMENT ON VIEW analysis.view_trajectories
IS 'GPS locations – Trajectories.';
```

In QGIS you can visualize the content of *analysis.view\_trajectories*.

Lastly, create one more view to spatially summarize the GPS data set using convex hull polygons (or minimum convex polygons):

```sql
CREATE VIEW analysis.view_convex_hulls AS
  SELECT 
    animals_id,
    (ST_ConvexHull(ST_Collect(geom)))::geometry(Polygon,4326) AS geom
  FROM 
    main.gps_data_animals 
  WHERE 
    geom IS NOT NULL 
  GROUP BY 
    animals_id 
  ORDER BY 
    animals_id; 
```

```sql
COMMENT ON VIEW analysis.view_convex_hulls
IS 'GPS locations - Minimum convex polygons.';
```

If you visualize this view in QGIS you can clearly see the effect of the outliers located far from  you can clearly see the effect of the outliers located far from the study area.

This last view is correct only if the GPS positions are located in a relatively small area (e.g. less than 50 kilometers) because the minimum convex polygon of points in geographic coordinates cannot be calculated assuming that coordinates are related to Euclidean space. At the moment the function *ST\_ConvexHull* does not support the GEOGRAPHY data type, so the correct way to proceed would be to project the GPS locations in a proper reference system, calculate the minimum convex polygon and then convert the result back to geographic coordinates. In the example, the error is negligible.

##### Exercise

1.  Create a view of all the points of female animals, visualize it in QGIS and export as shapefile
2.  Create a view with a convex hull for all the points of every month for animal 2 and visualize in QGIS to check if there is any spatial pattern
3.  Calculate the area of the monthly convex hulls of animal 2 and verify if there is any temporal pattern


## <a name="c_2.8"></a>2.8 Integrating spatial ancillary information



Animals move in and interact with complex environments that can be characterized by a set of spatial layers containing environmental data. Spatial databases can manage these different data sets in a unified framework, defining spatial and non-spatial relationships that simplify the analysis of the interaction between animals and their habitat. This simplifies a large set of analyses that can be performed directly in the database with no need for dedicated GIS or statistical software. Such an approach moves the information content managed in the database from a *geographical space* to an *animal's ecological space*. This more comprehensive database model of the animals' movement ecology reduces the distance between physical reality and the way data are structured in the database, filling the semantic gap between the scientist's view of biological systems and its implementation in the information system. This lesson shows how vector and raster layers can be included in the database and how you can handle them using (spatial) SQL. The database built so far is extended with environmental ancillary data sets.
 
### Adding ancillary environmental layers
In traditional information systems for wildlife tracking data management, position data are stored in some file-based spatial format (e.g. shapefile). With a multi-steps process in a GIS environment, position data are associated with a set of environmental attributes through an analytical stage (e.g. intersection of GPS positions with vector and raster environmental layers). This process is usually time-consuming and prone to error, implies data replication, and often has to be repeated for any new analysis. It also generally involves different tools for vector and raster maps. An advanced data management system should achieve the same result with an efficient (and, if needed, automated) procedure, possibly performed as a real-time routine management task. To do so, the first step is to integrate both position data and spatial ancillary information on the environment in a unique framework. This is essential to exploring the animals' behavior and understanding the ecological relationships that can be revealed by tracking data. Spatial databases can manage these different data sets in a unified framework. This also affects performance, as databases are optimized to run simple processes on large data sets like the ones generated by GPS sensors. In this exercise, you will see how to integrate a number of spatial features.

-   Points: meteorological stations (derived from [MeteoTrentino](http://www.meteotrentino.it/))
-   Linestrings: roads network (derived from [OpenStreetMap](http://www.openstreetmap.org/))
-   Polygons: administrative units (derived from [ISTAT](http://www.istat.it/it/strumenti/cartografia)) and the study area.
-   Rasters: land cover (source: [Corine](http://www.eea.europa.eu/data-and-maps/data/corine-land-cover-2006-clc2006-100-m-version-12-2009)) and digital elevation models (source: [SRTM](http://srtm.csi.cgiar.org/), see also *Jarvis A, Reuter HI, Nelson A, Guevara E (2008) Hole-filled seamless SRTM data V4. International Centre for Tropical Agriculture (CIAT)*).

Each species and study have specific data sets required and available, so the goal of this example is to show a complete set of procedures that can be replicated and customized on different data sets. Once layers are integrated into the database, you are encouraged to visualize and explore them in a GIS environment (e.g. QGIS). Once data are loaded into the database, you will extend the *gps\_data\_animals* table with the environmental attributes derived from the ancillary layers provided in the test data set. It is a good practice to store your environmental layers in a dedicated schema in order to keep a clear database structure. Let's create the schema *env\_data*:

```sql
CREATE SCHEMA env_data
  AUTHORIZATION postgres;
GRANT USAGE ON SCHEMA env_data TO basic_user;
```

```sql
COMMENT ON SCHEMA env_data 
IS 'Schema that stores environmental ancillary information.';
```

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA env_data 
  GRANT SELECT ON TABLES TO basic_user;
```

Now you can start importing the shapefiles of the (vector) environmental layers included in the test data set. An option is to use the drag and drop function of *DB Manager* (from QGIS Browser) plugin in QGIS (from the *QGIS Data Browser*), or the DB manager tool *Update/import layer* in QGIS desktop. 

Finish uploading the roads and administrative boundaries shapefiles using the 'Import vector layer' tool, using the same settings as for the study area shapefile.

Alternatively, a standard solution to import shapefiles (vector data) is the **[shp2pgsql](http://suite.opengeo.org/4.1/dataadmin/pgGettingStarted/shp2pgsql.html)** tool. *sph2pgsql* is an external command-line tool, which can not be run in a SQL interface as it can for a regular SQL command. The code below has to be run in a command-line interpreter (if you are using Windows as operating system, it is also called Command Prompt or MS-DOS shell). You will see other examples of external tools that are run in the same way, and it is very important to understand the difference between these and SQL commands. In these exercises, this difference is represented with a different graphic layout. Start with the meteorological stations:

```
> "C:\Program Files\PostgreSQL\9.5\bin\shp2pgsql.exe" -s 4326 -I C:\tracking_db\data\env_data\vector\meteo_stations.shp env_data.meteo_stations | "C:\Program Files\PostgreSQL\9.5\bin\psql.exe" -p 5432 -d gps_tracking_db -U postgres -h localhost
```

Note that the path to *shp2pgsql.exe* and *psql.exe* (in this case *C:\\PostgreSQL\\9.5\\bin)* can be different according to the folder where you installed your version of PostgreSQL. If you connect to the database remotely, you also have to change the address of the server (*-h* option). In the parameters, set the reference system (option -s) and create a spatial index for the new table (option *-I*). The result of *shp2pgsql* is a text file with the SQL that generates and populates the table *env\_data.meteo\_stations*. With the symbol '|' you 'pipe' (send directly) the SQL to the database (through the PostgreSQL interactive terminal **[psql](http://www.postgresql.org/docs/devel/static/app-psql.html))** where it is automatically executed. You have to set the the port (*-p*), the name of the database (*-d*), the user (*-U*) and the password, if requested. In this way, you complete the whole process with a single command. You can refer to *shp2pgsql* documentation for more details. You might have to add the whole path to *psql* and *shp2pgsql*. This depends on the folder where you installed PostgreSQL. You can easily verify the path searching for these two files. You also have to check that the path of your shapefile (*meteo\_stations.shp*) is properly defined. You can repeat the same operation for the study area layer:

```
> "C:\PostgreSQL\9.5\bin\shp2pgsql.exe" -s 4326 -I C:\tracking_db\data\env_data\vector\study_area.shp env_data.study_area | "C:\Program Files\PostgreSQL\9.5\bin\psql.exe" -p 5432 -d gps_tracking_db -U postgres -h localhost
```

Next for the roads layer:

```
> "C:\PostgreSQL\9.5\bin\shp2pgsql.exe" -s 4326 -I C:\tracking_db\data\env_data\vector\roads.shp env_data.roads | "C:\Program Files\PostgreSQL\9.5\bin\psql.exe" -p 5432 -d gps_tracking_db -U postgres -h localhost
```

And for the administrative boundaries:

```
> "C:\PostgreSQL\9.5\bin\shp2pgsql.exe" -s 4326 -I C:\tracking_db\data\env_data\vector\adm_boundaries.shp env_data.adm_boundaries | "C:\Program Files\PostgreSQL\9.5\bin\psql.exe" -p 5432 -d gps_tracking_db -U postgres -h localhost
```

Now the shapefiles are in the database as new tables (one table for each shapefile). You can visualize them through a GIS interface (e.g. QGIS). You can also retrieve a summary of the information from all vector layers available in the database with the following command:

```sql
SELECT * FROM geometry_columns;
```

The primary method to import a raster layer is the command-line tool **[raster2pgsql](http://postgis.net/docs/using_raster_dataman.html)**, the equivalent of shp2pgsql but for raster files, that converts GDAL-supported rasters into SQL suitable for loading into PostGIS. It is also capable of loading folders of raster files. **[GDAL](http://www.gdal.org/)** (Geospatial Data Abstraction Library) is a (free) library for reading, writing and processing raster geospatial data formats. It has a lot of simple but very powerful and fast command-line tools for raster data translation and processing. The related OGR library provides a similar capability for simple vector data features. GDAL is used by most of the spatial open-source tools and by a large number of commercial software programs as well. You will probably benefit in particular from the tools *gdalinfo* (get a layer's basic metadata), *gdal\_translate* (change data format, change data type, cut), gdalwarp1 (mosaicing, reprojection and warping utility).

An interesting feature of *raster2pgsql* is its capability to store the rasters inside the database (in-db) or keep them as (out-db) files in the file system (with the raster2pgsql -R option). In the last case, only raster metadata are stored in the database, not pixel values themselves. Loading out-db rasters metadata is much faster than loading them completely in the database. Most operations at the pixel values level (e.g. \*ST\_SummaryStats\*) will have equivalent performance with out- and in-db rasters. Other functions, like *ST\_Tile*, involving only the metadata, will be faster with out-db rasters. Another advantage of out-db rasters is that they stay accessible for external applications unable to query databases (with SQL). However, the administrator must make sure that the link between what is in the db (the path to the raster file in the file system) is not broken (e.g. by moving or renaming the files). On the other hand, only in-db rasters can be generated with CREATE TABLE and modified with UPDATE statements. Which is the best choice depends on the size of the data set and on considerations about performance and database management. A good practice is generally to load very large raster data sets as out-db and to load smaller ones as in-db to save time on loading and to avoid repeatedly backing up huge, static rasters.

The QGIS plugin *Load Raster to PostGIS* can also be used to import raster data with a graphical interface. An important parameter to set when importing raster layers is the number of tiles (*-t* option). Tiles are small subsets of the image and correspond to a physical record in the table. This approach dramatically decreases the time required to retrieve information. The recommended values for the tile option range from 20x20 to 100x100. Here is the code (to be run in the Command Prompt) to transform a raster (the digital elevation model derived from SRTM) into the SQL code that is then used to physically load the raster into the database (as you did with *shp2pgsql* for vectors):

```
> "C:\PostgreSQL\9.6\bin\raster2pgsql.exe" -I -M -C -s 4326 -t 20x20 C:\tracking_db\data\env_data\raster\srtm_dem.tif env_data.srtm_dem | "C:\Program Files\PostgreSQL\9.6\bin\psql.exe" -p 5432 -d gps_tracking_db -U postgres -h localhost
```

If you copy-paste the copy from an Internet browser, some character (e.g. double quotes and 'x') might be transformed into different character and you might have to manually fix this problem). You can repeat the same process on the land cover layer:

```
> "C:\PostgreSQL\9.6\bin\raster2pgsql.exe" -I -M -C -s 3035 C:\tracking_db\data\env_data\raster\corine06.tif -t 20x20 env_data.corine_land_cover | "C:\Program Files\PostgreSQL\9.6\bin\psql.exe" -p 5432 -d gps_tracking_db -U postgres -h localhost
```

The reference system of the Corine Land Cover data set is not geographic coordinates (SRID 4326), but ETRS89/ETRS-LAEA (SRID 3035), an equal-area projection over Europe. This must be specified with the -s option and kept in mind when this layer will be connected to other spatial layers stored in a different reference system. As with shp2pgsql.exe, the -I option will create a spatial index on the loaded tiles, speeding up many spatial operations, and the*-C* option will generate a set of constraints on the table, allowing it to be correctly listed in the *raster\_columns* metadata table. The land cover raster identifies classes that are labeled by a code (an integer). To specify the meaning of the codes, you can add a table where they are described. In this example, the land cover layer is taken from the Corine project. Classes are described by a hierarchical legend over three nested levels. The legend is provided in the test data set in the file *corine\_legend.csv*. You import the table of the legend (first creating an empty table, and then loading the data):

```sql
CREATE TABLE env_data.corine_land_cover_legend(
  grid_code integer NOT NULL,
  clc_l3_code character(3),
  label1 character varying,
  label2 character varying,
  label3 character varying,
  CONSTRAINT corine_land_cover_legend_pkey 
    PRIMARY KEY (grid_code ));
```

```sql
COMMENT ON TABLE env_data.corine_land_cover_legend
IS 'Legend of Corine land cover, associating the numeric code to the three nested levels.';
```

Then you load the data:

```sql
COPY env_data.corine_land_cover_legend 
FROM 
  'C:\tracking_db\data\env_data\raster\corine_legend.csv' 
  WITH (FORMAT csv, HEADER, DELIMITER ';');
```

You can retrieve a summary of the information from all raster layers available in the database with the following command:

```sql
SELECT * FROM raster_columns;
```

To keep a well-documented database, add comments to describe all the spatial layers that you have added:

```sql
COMMENT ON TABLE env_data.adm_boundaries 
IS 'Layer (polygons) of administrative boundaries (comuni).';
```

```sql
COMMENT ON TABLE env_data.corine_land_cover 
IS 'Layer (raster) of land cover (from Corine project).';
```

```sql
COMMENT ON TABLE env_data.meteo_stations 
IS 'Layer (points) of meteo stations.';
```

```sql
COMMENT ON TABLE env_data.roads 
IS 'Layer (lines) of roads network.';
```

```sql
COMMENT ON TABLE env_data.srtm_dem 
IS 'Layer (raster) of digital elevation model (from SRTM project).';
```

```sql
COMMENT ON TABLE env_data.study_area 
IS 'Layer (polygons) of the boundaries of the study area.';
```

### Playing with spatial SQL

As the set of ancillary (spatial) information is now loaded into the database, you can start playing with this information using spatial SQL queries. In fact, it is possible with spatial SQL to run queries that explicitly handle the spatial relationships among the different spatial tables that you have stored in the database. In the following examples, SQL statements will show you how to take advantage of PostGIS features to manage, explore and analyze spatial objects, with optimized performances and no need for specific GIS interfaces.

You start playing with your spatial data by asking for the name of the administrative unit (*comune*, Italian commune) in which the point at coordinates (11, 46) (longitude, latitude) is located. There are two commands that are used when it comes to intersection of spatial elements: *ST\_Intersects* and *ST\_Intersection*. The former returns true if two features intersect, while the latter returns the geometry produced by the intersection of the objects. In this case, *ST\_Intersects* is used to select the right comune:

```sql
SELECT 
  nome_com
FROM 
  env_data.adm_boundaries 
WHERE 
  ST_Intersects((ST_SetSRID(ST_MakePoint(11,46), 4326)), geom);
```
In the second example, you compute the distance (rounded to the meter) from the point at coordinates (11, 46) to all the meteorological stations (ordered by distance) in the table *env\_data.meteo\_stations*. This information could be used, for example, to derive the precipitation and temperature for a GPS position at the given acquisition time, weighting the measurement from each station according to the distance to the point. In this case, *ST\_DistanceSpheroid* is used. Alternatively, you could use *ST\_Distance* and cast your geometries as *geography* data types.

```sql
SELECT 
  station_id, ST_DistanceSpheroid((ST_SetSRID(ST_MakePoint(11,46), 4326)), geom, 'SPHEROID["WGS 84",6378137,298.257223563]')::integer AS distance
FROM 
  env_data.meteo_stations
ORDER BY 
  distance;
```

In the third example, you compute the distance to the closest road:

```sql
SELECT 
  ST_Distance((ST_SetSRID(ST_MakePoint(11,46), 4326))::geography, geom::geography)::integer AS distance
FROM 
  env_data.roads
ORDER BY 
  distance 
LIMIT 1;
```

For users, the data type (vector, raster) used to store spatial information is not so relevant when they query their data: queries should transparently use any kind of spatial data as input. Users can then focus on the environmental model instead of worrying about the data model. In the next example, you intersect a point with two raster layers (altitude and land cover) in the same way you do for vector layers. In the case of land cover, the point must first be projected into the Corine reference system (SRID 3035). In the raster layer, just the Corine code class (integer) is stored while the legend is stored in the table *env\_data.corine\_land\_cover\_legend*. In the query, the code class is joined to the legend table and the code description is returned. This is an example of integration of both spatial and non-spatial elements in the same query.

```sql
SELECT 
  ST_Value(srtm_dem.rast,
  (ST_SetSRID(ST_MakePoint(11,46), 4326))) AS altitude,
  ST_value(corine_land_cover.rast,
  ST_transform((ST_SetSRID(ST_MakePoint(11,46), 4326)), 3035)) AS land_cover, 
  label2, 
  label3
FROM 
  env_data.corine_land_cover, 
  env_data.srtm_dem, 
  env_data.corine_land_cover_legend
WHERE 
  ST_Intersects(
    corine_land_cover.rast,
    ST_Transform((ST_SetSRID(ST_MakePoint(11,46), 4326)), 3035)) AND
  ST_Intersects(srtm_dem.rast,(ST_SetSRID(ST_MakePoint(11,46), 4326))) AND
  grid_code = ST_Value(
    corine_land_cover.rast,
    ST_Transform((ST_SetSRID(ST_MakePoint(11,46), 4326)), 3035));
```

Now combine roads and administrative boundaries to compute how many meters of roads there are in each administrative unit. You first have to intersect the two layers (*ST\_Intersection*), then compute the length (*ST\_Length*) and summarize per administrative unit (sum() associated with GROUP BY clause).

```sql
SELECT 
  nome_com, 
  sum(ST_Length(
    (ST_Intersection(roads.geom, adm_boundaries.geom))::geography))::integer AS total_length
FROM 
  env_data.roads, 
  env_data.adm_boundaries 
WHERE 
  ST_Intersects(roads.geom, adm_boundaries.geom)
GROUP BY 
  nome_com 
ORDER BY 
  total_length desc;
```

The last examples are about the interaction between rasters and polygons. In this case, we compute some statistics (minimum, maximum, mean, and standard deviation) for the altitude within the study area:

```sql
SELECT 
  (sum(ST_Area(((gv).geom)::geography)))/1000000 area,
  min((gv).val) alt_min, 
  max((gv).val) alt_max,
  avg((gv).val) alt_avg,
  stddev((gv).val) alt_stddev
FROM
  (SELECT 
    ST_intersection(rast, geom) AS gv
  FROM 
    env_data.srtm_dem,
    env_data.study_area 
  WHERE 
    ST_intersects(rast, geom)
) foo;
```

The result show the large variability of altitude across the study area.

You might also be interested in the number of pixels of each land cover type within the study area. As with the previous example, we first intersect the study area with the raster of interest, but in this case we need to reproject the study area polygon into the coordinate system of the Corine land cover raster (SRID: 3035). With the following query, you can see the dominance of mixed forests in the study area:

```sql
SELECT (pvc).value, SUM((pvc).count) AS total, label3
FROM 
  (SELECT ST_ValueCount(rast) AS pvc
  FROM env_data.corine_land_cover, env_data.study_area
  WHERE ST_Intersects(rast, ST_Transform(geom, 3035))) AS cnts, 
  env_data.corine_land_cover_legend
WHERE grid_code = (pvc).value
GROUP BY (pvc).value, label3
ORDER BY (pvc).value;
```

The previous query can be modified to return the percentage of each class over the total number of pixels. This can be achieved using **[window functions](http://www.postgresql.org/docs/devel/static/tutorial-window.html)**:

```sql
SELECT 
  (pvc).value, 
  (SUM((pvc).count)*100/
    SUM(SUM((pvc).count)) over ()
  )::numeric(4,2) AS total_perc, label3
FROM 
  (SELECT ST_ValueCount(rast) AS pvc
  FROM env_data.corine_land_cover, env_data.study_area
  WHERE ST_Intersects(rast, ST_Transform(geom, 3035))) AS cnts, 
  env_data.corine_land_cover_legend
WHERE grid_code = (pvc).value
GROUP BY (pvc).value, label3
ORDER BY (pvc).value;
```

##### Exercise

1.  What is the administrative unit where each meteo station is located?
2.  What is the land cover class where each meteo station is located?
3.  What is the distance of each GPS position to the closest road?
4.  What is the proportion of GPS locations in each land cover class used by all animals?







## <a name="c_2.9"></a>2.9 Data quality: how to detect and manage outliers

























## <a name="c_2.10"></a>2.10 Data export 
There are different ways to export a table or the results of a query to an external file. One is to use the command **[COPY (TO)](http://www.postgresql.org/docs/devel/static/sql-copy.html)**. `COPY TO` (similarly to what happens with the command `COPY FROM` used to import data) with a file name directly write the content of a table or the result of a query to a file, for example in .csv format. The file must be accessible by the PostgreSQL user (i.e. you have to check the permission on target folder by the user ID the PostgreSQL server runs as) and the name (path) must be specified from the viewpoint of the server. This means that files can be read or write only in folders 'visible' to the database servers. If you want to remotely connect to the database and save data into your local machine, you should use the command **[\COPY](http://www.postgresql.org/docs/devel/static/app-psql.html#APP-PSQL-META-COMMANDS-COPY)** instead. It performs a frontend (client) copy. `\COPY` is not an SQL command and must be run from a PostgreSQL interactive terminal **[PSQL](http://www.postgresql.org/docs/devel/static/app-psql.html)**. This is an operation that runs an SQL COPY command, but instead of the server reading or writing the specified file, PSQL reads or writes the file and routes the data between the server and the local file system. This means that file accessibility and privileges are those of the local user, not the server, and no SQL superuser privileges are required. 
Another possibility to export data is to use the pgAdmin interface: in the SQL console select `Query/Execute to file`, the results will be saved to a local file instead of being visualized. Other database interfaces have similar tools. This can be applied to any query.
For spatial data, the easiest option is to load the data in QGIS and then save as shapefile (or any other format) on your computer.

## <a name="c_2.11"></a>2.11 Database maintenance
Once your database is populated and used for daily work, it is a *really* good idea to routinely make a safe copy of your data. Since the RDBMS maintains data in a binary format which is not meant to be tampered with, we need to `dump` the database content in a format suitable for being later restored if it needs be. The very same dump could also be used for replicating the database contents on another server.
From pgAdmin, the operation of making a database dump is extremely simple: right click the database and choose `Backup`.
There are a few output formats, apart from the default `Custom` one. With `Plain` the file will be plain (readable) SQL commands that can be opened (and edit, if needed) with a text editor (e.g. Notepad++). `Tar` will generate a compressed file that is convenient if you have frequent backups and you want to maintain an archive. For more info, see the docs on **[backup and restore](http://www.postgresql.org/docs/current/static/backup-dump.html)** for further information.  
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

## <a name="c_2.12"></a>2.12 Recap exercises

1.  Calculate number of locations per animal per month
2.  Calculate average distance per animal per month
3.  Find animals at same place at the same time
4.  Find locations of an animal in the home range (convex hulls) of another animal
5.  Calculate percentage of location per land cover type per animal