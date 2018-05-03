# 1. SQL and Spatial SQL
## 1.1 WORKING WITH SQL
This lesson introduces students to SQL and illustrates the main commands that are needed to interact with a database. Each command is described then an example shows how it works. At the end, an exercise is used to let students experiment it by themselves. At the end of the lesson, you will be able to get data from the database specifying criteria to retrieve the desired subset of records.

### 1.1.1 Introduction to SQL
The state-of-the-art technical tool for effectively and efficiently managing movement and population ecology data is the spatial relational database management systems (*SRDBMS* for short). Using databases to manage data implies a considerable effort for those who are not already familiar with these tools, but this is necessary to be able to deal with the large and/or complex data sets in a multi-user context where errors in data might be critical and data can be re-used many times for different projects, including in the long term. Moreover, the time spent to learn databases will be largely paid back with the time saved for the management and processing of the data. 

*SQL*, which stands for *Structured Query Language* is a way for interacting with SRDBMS. SQL statements are used to perform tasks such as update data on a database, create database object or retrieve data from a database. In this lesson we will focus on data retrieval from an existing database. SQL is highly standardized and while each database platform has some kind of SQL dialect, once learnt SQL can be used with any SRDBMS tool (e.g. PostgreSQL [used during this course], MySQL, ORACLE, SQLServer, SQLite, SpatiaLite). While complex queries can be hard to design, SQL itself is a simple language that combines a very limited sets of commands in a way that is similar to the natural language.

The reference software platform used is the open source PostgreSQL with its spatial extension PostGIS. The reference (graphical) interface used to deal with the database
is [pgAdmin3](http://www.pgadmin.org/). All the examples provided (SQL code) and technical solutions proposed are tuned on this software, although most of the code can be easily adapted for other platforms.



### 1.1.2 Overview of the database used for the exercises
[...]
data content
connection parameters

### 1.1.3 Schemas, tables, data types
The basic structure of a database is called a *table*. As you would expect it is composed of columns and rows, but unlike what happens in Excel or Calc, you cannot put whatever you want in it. A table is declaratively created with a structure: each column has a defined *type* of data, and the rows (also called *records*) must respect this type: the system enforces this constraint, and does not allow the wrong kind of data to slip in. Each data type has specific properties and functions associated. You will see how to create a table in sections 2 and 3, when you will create your own database.

Tables can be linked to one another (the jargon term for this kind of link is *relation*, which accounts for the *R* in *RDBMS*): you can explicitly ask that the value to put in a specific record column comes from another table. This helps reduce data replication, and explicitly keeps track of inter-table structure in a formalized way.

[SCHEMA](http://www.postgresql.org/docs/devel/static/sql-createschema.html)
[TABLE](https://www.postgresql.org/docs/devel/static/sql-createtable.html)
[DATA TYPES](http://www.postgresql.org/docs/devel/static/datatype.html)

[...]

> Inside our new empty database, we would like to create tables -
> possibly grouped together into logical units called
> [schemas](http://www.postgresql.org/docs/9.4/interactive/ddl-schemas.html).
> A table in a relational database is much like a table on paper: It
> consists of rows and columns. The number and order of the columns is
> fixed, and each column has a name. The number of rows is variable — it
> reflects how much data is stored at a given moment. SQL does not make
> any guarantees about the order of the rows in a table. When a table is
> read, the rows will appear in an unspecified order, unless sorting is
> explicitly requested. Furthermore, SQL does not assign unique
> identifiers to rows, so it is possible to have several completely
> identical rows in a table. This is a consequence of the mathematical
> model that underlies SQL but is usually not desirable. We will see how
> to deal with this issue in a while.
> 
> Each column has a data type. The data type constrains the set of
> possible values that can be assigned to a column and assigns semantics
> to the data stored in the column so that it can be used for
> computations. For instance, a column declared to be of a numerical
> type will not accept arbitrary text strings, and the data stored in
> such a column can be used for mathematical computations. By contrast,
> a column declared to be of a character string type will accept almost
> any kind of data but it does not lend itself to mathematical
> calculations, although other operations such as string concatenation
> are available.
> 
> PostgreSQL includes a sizable set of built-in data types that fit many
> applications. Users can also define their own data types. Most
> built-in data types have obvious names and semantics, and are
> explained in great detail in PostgreSQL
> [datatype documentation](http://www.postgresql.org/docs/9.4/interactive/datatype.html).
> Some of the frequently used data types are: `integer` for whole
> numbers, `numeric` for possibly fractional numbers, `text` for
> character strings, `date` for dates, `time` for time-of-day values,
> and `timestamp` for values containing both date and time.

*some tricks on the use of pgadmin?*

### 1.1.4 SELECT, FROM, WHERE
The operation of choosing the records you want is called *selection*: the `SELECT` command allows you to express clearly which columns you need, which rows, and in which
order; you can also generate computed data on the fly. The basic structure of a `SELECT` command is the following:

```sql
SELECT
   <one or more columns, or * for all>
FROM
   <one or more tables>
WHERE
   <conditions to filter retrieved records on>
;
```

SQL is case insensitive. SQL statements can be long and span over multiple lines; to signal to the database server the they are complete and should be executed, you have to terminate them with a semicolon `;`.

`SELECT` command can be used without any other commands. Here some examples.

```sql
SELECT 1+1;
```

```sql
SELECT 'Hi there';
```

```sql
SELECT now();
```

`FROM` command specifies the tables where is stored the required information.

[example with set of columns, example with all columns]
```sql
xxxx
```

```sql
xxxx
```

`WHERE` is used to set criteria on the data that you want to retrieve.

[example with 1 criteria on number and 1 criteria on string]
```sql
xxxx
```

```sql
xxxx
```

The complete reference of the `SELECT` statement and related commands is available
[here](https://www.postgresql.org/docs/devel/static/sql-select.html).

### 1.1.5 AND, OR, IN, !=, NULL
### 1.1.6 ORDER BY, LIMIT, DISTINCT
Sometimes we are only interested in which values do appear, and not on
specific records. In this casee would use `DISTINCT` to squash duplicate values, like so:

```sql
SELECT DISTINCT ccc FROM www;
```

### 1.1.7 LIKE
### 1.1.8 GROUP BY (COUNT, SUM, MIN, MAX, AVG, STDDEV)
### 1.1.9 HAVING
### 1.1.10 Joining multiple tables
### 1.1.11 LEFT JOIN
### 1.1.12 Nested queries
### 1.1.13 WINDOW functions
### 1.1.14 INSERT, UPDATE, DELETE
> Data modification in SQL is accomplished with three statements:
> `INSERT`, `UPDATE`, `DELETE`. Syntax is pretty simple, let's see a few
> examples:
> 
> ```sql
> INSERT INTO main.animals (animals_id,animals_code,name,sex) VALUES (7,'NEW01','new','f');
> ```
> 
> Check what's happened:
> 
> ```sql
> SELECT * FROM main.animals WHERE animals_id=7;
> ```
> 
> ```
>  animals_id | animals_code | name | sex | age_class_code | species_code | note |       insert_timestamp        |       update_timestamp        
> ------------+--------------+------+-----+----------------+--------------+------+-------------------------------+-------------------------------
>           7 | NEW01        | new  | f   |                |              |      | 2015-06-25 10:58:38.655102+02 | 2015-06-25 10:58:38.655102+02
> (1 row)
> ```
> 
> We forgot to define `age_class_code` and `species_code`, so let's add
> them to our new record:
> 
> ```sql
> UPDATE main.animals
> SET age_class_code = 1, species_code = 3
> WHERE animals_id = 7;
> ```
> 
> Pay attention to `UPDATE` statements: if you forget the `WHERE` part,
> they apply to the whole table - in the present case clearly not what
> we want.
> 
> ```sql
> SELECT * FROM main.animals WHERE animals_id=7;
> ```
> 
> ```
>  animals_id | animals_code | name | sex | age_class_code | species_code | note |       insert_timestamp        |       update_timestamp        
> ------------+--------------+------+-----+----------------+--------------+------+-------------------------------+-------------------------------
>           7 | NEW01        | new  | f   |              1 |            3 |      | 2015-06-25 10:58:38.655102+02 | 2015-06-25 11:02:42.463723+02
> (1 row)
> ```
> 
> To get rid of the test record we just added:
> 
> ```sql
> DELETE FROM main.animals WHERE animals_id=7;
> ```
> 
> Here too you need to remember the `WHERE` clause: otherwise all of
> your records will be deleted!

## 1.2 WORKING WITH SPATIO-TEMPORAL SQL
### 1.2.1 Temporal data (date, time, timezone), EXTRACT

PostgreSQL (and most of the database systems) can deal with a large
set of specific type of data with dedicated database data types and
related functionalities in addition to string and numbers. In the next
exercise, a new field is introduced to represent a
[timestamp with time zone](http://www.postgresql.org/docs/9.1/static/datatype-datetime.html),
i.e. date + time + time zone together (later on in this tutorial,
another specific data type will be introduced to manage spatial data).

### 1.2.2 Spatial objects in PostGIS (points, lines, polygons, raster)
### 1.2.3 Visualize the coordinates of a spatial object
### 1.2.4 Reference systems and projections 
### 1.2.5 Create a point from coordinates
### 1.2.6 Create a line from ordered points (trajectory)
### 1.2.7 Calculate length of a trajectory
### 1.2.8 Create a polygon from points (convex hull)
### 1.2.9 Calculate the area of a polygon
### 1.2.10 Visualize spatial data in QGIS
