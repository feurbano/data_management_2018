# 1. SQL and Spatial SQL

* [1.1  Introduction to SQL](#c_1.1)
* [1.2  Overview of the database used for the exercises](#c_1.2)
* [1.3  Schemas, tables, data types](#c_1.3)
* [1.4  SELECT, FROM, WHERE](#c_1.4)
* [1.5  AND, OR, IN, !=, NULL](#c_1.5)
* [1.6  ORDER BY, LIMIT, DISTINCT, CASE, CAST](#c_1.6)
* [1.7  LIKE](#c_1.7)
* [1.8  GROUP BY (COUNT, SUM, MIN, MAX, AVG, STDDEV)](#c_1.8)
* [1.9  HAVING](#c_1.9)
* [1.10  Joining multiple tables](#c_1.10)
* [1.11  LEFT JOIN](#c_1.11)
* [1.12  Nested queries](#c_1.12)
* [1.13  WINDOW functions](#c_1.13)
* [1.14  INSERT, UPDATE, DELETE](#c_1.14)
* [1.15  Temporal data (date, time, timezone), EXTRACT](#c_1.15)
* [1.16  Spatial objects in PostGIS](#c_1.16)
* [1.17  Visualize the coordinates of a spatial object](#c_1.17)
* [1.18  Reference systems and projections](#c_1.18)
* [1.19  Create a point from coordinates](#c_1.19)
* [1.2  Create a line from ordered points (trajectory)](#c_1.2)
* [1.21  Calculate length of a trajectory](#c_1.21)
* [1.22  Create a polygon from points (convex hull)](#c_1.22)
* [1.23  Calculate the area of a polygon](#c_1.23)
* [1.24  Visualize spatial data in QGIS](#c_1.24)


## <a name="c_1.1"></a>1.1 Introduction to SQL
The state-of-the-art technical tool for effectively and efficiently managing movement and population ecology data is the spatial relational database management systems (*SRDBMS* for short). Using databases to manage data implies a considerable effort for those who are not already familiar with these tools, but this is necessary to be able to deal with the large and/or complex data sets in a multi-user context where errors in data might be critical and data can be re-used many times for different projects, including in the long term. Moreover, the time spent to learn databases will be largely paid back with the time saved for the management and processing of the data. 

*SQL*, which stands for *Structured Query Language* is a way for interacting with SRDBMS. SQL statements are used to perform tasks such as update data on a database, create database object or retrieve data from a database. In this lesson we will focus on data retrieval from an existing database. SQL is highly standardized and while each database platform has some kind of SQL dialect, once learnt SQL can be used with any SRDBMS tool (e.g. PostgreSQL [used during this course], MySQL, ORACLE, SQLServer, SQLite, SpatiaLite). While complex queries can be hard to design, SQL itself is a simple language that combines a very limited sets of commands in a way that is similar to the natural language.

The reference software platform used is the open source PostgreSQL with its spatial extension PostGIS. The reference (graphical) interface used to deal with the database
is [pgAdmin3](http://www.pgadmin.org/). All the examples provided (SQL code) and technical solutions proposed are tuned on this software, although most of the code can be easily adapted for other platforms.

This lesson introduces students to SQL and spatial SQL and illustrates the main commands that are needed to interact with a database. Each command is described then an example shows how it works. At the end, an exercise is used to let students experiment by themselves.

## <a name="c_1.2"></a>1.2 Overview of the database used for the exercises
[...]
data content
connection parameters

## <a name="c_1.3"></a>1.3 Schemas, tables, data types
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

## <a name="c_1.4"></a>1.4 SELECT, FROM, WHERE
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

## <a name="c_1.5"></a>1.5 AND, OR, IN, !=, NULL
## <a name="c_1.6"></a>ORDER BY, LIMIT, DISTINCT, CASE, CAST
Sometimes we are only interested in which values do appear, and not on
specific records. In this casee would use `DISTINCT` to squash duplicate values, like so:

```sql
SELECT DISTINCT ccc FROM www;
```

## <a name="c_1.7"></a>1.7 LIKE
## <a name="c_1.8"></a>1.8 GROUP BY (COUNT, SUM, MIN, MAX, AVG, STDDEV)
## <a name="c_1.9"></a>1.9 HAVING
## <a name="c_1.10"></a>1.10 Joining multiple tables
## <a name="c_1.11"></a>1.11 LEFT JOIN
## <a name="c_1.12"></a>1.12 Nested queries
## <a name="c_1.13"></a>1.13 WINDOW functions

A window function performs a calculation across a set of rows that are somehow related to the current row. This is similar to an aggregate function, but unlike regular aggregate functions, window functions do not group rows into a single output row, hence they are still able to access more than just the current row of the query result. In particular, it enables you to access previous and next rows (according to a user-defined ordering criteria) while calculating values for the current row. This is very useful, as a tracking data set has a predetermined temporal order, where many properties (e.g. geometric parameters of the trajectory, such as turning angle and speed) involve a sequence of GPS positions. It is important to remember that the order of records in a database is irrelevant. The ordering criteria must be set in the query that retrieves data.

## <a name="c_1.14"></a>1.14 INSERT, UPDATE, DELETE
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
> To get rid of the test record we just added:
> 
> ```sql
> DELETE FROM main.animals WHERE animals_id=7;
> ```
> 
> Here too you need to remember the `WHERE` clause: otherwise all of
> your records will be deleted!

## <a name="c_1.15"></a>1.15 Temporal data (date, time, timezone), EXTRACT

PostgreSQL (and most of the database systems) can deal with a large
set of specific type of data with dedicated database data types and
related functionalities in addition to string and numbers. In the next
exercise, a new field is introduced to represent a
[timestamp with time zone](http://www.postgresql.org/docs/9.1/static/datatype-datetime.html),
i.e. date + time + time zone together (later on in this tutorial,
another specific data type will be introduced to manage spatial data).

## <a name="c_1.16"></a>1.16 Spatial objects in PostGIS


Until few years ago, the spatial information produced by GPS sensors was managed and analyzed using dedicated software (GIS) in file-based data formats (e.g. shapefile). Nowadays, the most advanced approaches in data management consider the spatial component of objects (e.g. a moving animal) as one of its many attributes: thus, while understanding the spatial nature of your data is essential to proper analysis, from a software perspective spatial is (less and less) not special. Spatial databases are the technical tool needed to implement this perspective. They integrate spatial data types (vector and raster) together with standard data types that store the objects' other (non-spatial) associated attributes. Spatial data types can be manipulated by SQL through additional commands and functions for the spatial domain. This possibility essentially allows you to build a GIS using the existing capabilities of relational databases. Moreover, while dedicated GIS software is usually focused on analyses and data visualization, providing a rich set of spatial operations, few are optimized for managing large spatial data sets (in particular, vector data) and complex data structures. Spatial databases, in turn, allow both advanced management and spatial operations that can be efficiently undertaken on a large set of elements. This combination of features is becoming essential, as with animal movement data sets the challenge is now on the extraction of synthetic information from very large data sets rather than on the extrapolation of new information (e.g. kernel home ranges from VHF data) from limited data sets with complex algorithms. 

--------------


This lesson will focus on introducing spatial data types, which will
enable you to accomplish richer analysis of wildlife tracking data. By
the end of the lesson you will become familiar with geometry columns,
and make the database compute answers to spatial questions. 

An ordinary database has strings, numbers, and dates. A spatial
database adds additional (spatial) types for representing geographic
features.  These spatial data types abstract and encapsulate spatial
structures such as boundary and dimension. In many respects, spatial
data types can be understood simply as shapes: typically points,
curves, surfaces and collections of them.
Such data were traditionally manipulated outside of databases using
specialized tools (GIS software). In the last 15
years or so, spurred by the wide usage of GPS systems, a few
implementations of GIS tools for RBMS have emerged. There has also
been an effort to [standardize](http://www.opengeospatial.org/) many
aspects of spatial systems, which made data exchange between different
platforms somewhat more comfortable.

> spatial is not special

For manipulating data during a query, an ordinary database provides
functions such as concatenating strings, performing hash operations on
strings, doing mathematics on numbers, and extracting information from
dates. A spatial database provides a complete set of functions for
analyzing geometric components, determining spatial relationships, and
manipulating geometries. These spatial functions serve as the building
block for any spatial project.

The majority of all spatial functions can be grouped into one of the
following five categories:

-   Conversion: Functions that convert between geometries and external
    data formats.
-   Management: Functions that manage information about spatial tables
    and PostGIS administration.
-   Retrieval: Functions that retrieve properties and measurements of
    a Geometry.
-   Comparison: Functions that compare two geometries with respect to
    their spatial relation.
-   Generation: Functions that generate new geometries from others.

The list of possible functions is very large, but a common set of
functions is defined by the
[OGC SFSQL](http://workshops.boundlessgeo.com/postgis-intro/glossary.html#term-sfsql).

In the Open Source world, one of the richest implementations of the
spatial SQL standards is provided by the
[PostGIS](http://postgis.net/) extension for PostgreSQL - and that was
one strong motivation for choosing this particular RDBMS for analyzing
tracking data.

As we have seen before, RDBMS allow for storing and searching large
amounts of data: to optimize access times, they make use of indexes
which are often in the form of
[B-trees](http://en.wikipedia.org/wiki/B-tree). Spatial data require a
different kind of indexes for efficient searching: spatial indexes are
generally computed around the concept of *bounding box*: A bounding
box is the smallest rectangle - parallel to the coordinate axes -
capable of containing a given feature.

Bounding boxes are used because answering the question “is A inside
B?”  is very computationally intensive for polygons but very fast in
the case of rectangles. Even the most complex polygons and linestrings
can be represented by a simple bounding box.

Indexes have to perform quickly in order to be useful. So instead of
providing exact results, as B-trees do, spatial indexes provide
approximate results. The question “what lines are inside this
polygon?”  will be instead interpreted by a spatial index as “what
lines have bounding boxes that are contained inside this polygon’s
bounding box?”

> raster

------------

> to be redistributed

------------

We have already enabled the spatial extensions for `demo`. The command
to install PostGIS inside a database is:

```sql
CREATE EXTENSION postgis;
```

To check that everything is in order, we can call our first PostGIS
function, `postgis_full_version`. On the server, we obtain the
following answer:

``` sql
SELECT postgis_full_version();
```

So far, so good. Now, we build a new `geometries` table with a column
of type (surprise, surprise!) `geometry`, and then put some data in
it:

```sql 
--don't run
CREATE TABLE geometries (name varchar, geom geometry);
INSERT INTO geometries VALUES
  ('Point', 'POINT(0 0)'),
  ('Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
  ('Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
  ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
  ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');
```

Note that we are inserting a string value into column `geom`: the
format is called
[Well-known text](https://en.wikipedia.org/wiki/Well-known_text), WKT
for short, and is also standardized.

The internal database representation is not meant for being readable:

``` sql
SELECT geom FROM geometries WHERE name = 'Point';
```



Among the many functions provided by PostGIS, you find one for
converting from the internal representation back to WKT:

```sql
SELECT name, ST_AsText(geom) FROM geometries;
```

```
      name       |                           st_astext                           
-----------------+---------------------------------------------------------------
 Point           | POINT(0 0)
 Linestring      | LINESTRING(0 0,1 1,2 1,2 2)
 Polygon         | POLYGON((0 0,1 0,1 1,0 1,0 0))
 PolygonWithHole | POLYGON((0 0,10 0,10 10,0 10,0 0),(1 1,1 2,2 2,2 1,1 1))
 Collection      | GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0,1 0,1 1,0 1,0 0)))
(5 rows)
```

Geometric data can also be computed directly from coordinates, as in:

```sql
SELECT ST_MakePoint(11.001,46.001) AS point;
```


The textual representation is easier to recognize, of course:

```sql
SELECT ST_AsText(ST_MakePoint(11.001,46.001)) AS point;
```

In conformance with the standard, PostGIS offers a way to track and
report on the geometry types available in a given database:

```sql
SELECT * FROM geometry_columns;
```


The previous query informs us that inside our `demo` there are 16
tables with a geometry column, and that each of these columns is named
`geom` and contains 2-dimensional data. Most of them, unlike the one
we have just created, only accept a specific data type. Furthermore,
they also carry information about the reference coordinate system in
use, via the [SRID](https://en.wikipedia.org/wiki/SRID) parameter.

The number `4326` refers to
[EPSG:4326](http://spatialreference.org/ref/epsg/4326/), which is
European Petroleum Survey Group identifier for WGS84:

```sql
GEOGCS["WGS 84",
    DATUM["WGS_1984",
        SPHEROID["WGS 84",6378137,298.257223563,
            AUTHORITY["EPSG","7030"]],
        AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0,
        AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.01745329251994328,
        AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4326"]]
```

In the case of `demo` data, we are not storing planar Euclidean
coordinates, but use latitude and longitude to identify a point on the
ellipsoid expressed by the geodetic datum WGS\_1984 - the one used
globally by GPS systems.

Our test table `geometries` actually does not specify an SRID, but we
can easily fix that:

```sql
SELECT UpdateGeometrySRID('geometries','geom',4326);
```

```
             updategeometrysrid              
---------------------------------------------
 public.geometries.geom SRID changed to 4326
(1 row)
```

Let's query the contents of our `geometries` table using PostGIS some
functions intended for metadata retrieval:

```sql
SELECT name, ST_GeometryType(geom), ST_NDims(geom), ST_SRID(geom)
FROM geometries;
```

```
      name       |    st_geometrytype    | st_ndims | st_srid 
-----------------+-----------------------+----------+---------
 Point           | ST_Point              |        2 |    4326
 Linestring      | ST_LineString         |        2 |    4326
 Polygon         | ST_Polygon            |        2 |    4326
 PolygonWithHole | ST_Polygon            |        2 |    4326
 Collection      | ST_GeometryCollection |        2 |    4326
(5 rows)
```

When using real world spatial data obtained from various sources, you
will likely encounter different coordinate systems. One of the tasks
that you will need to accomplish will be to re-project the data into a
common SRID, in order to be able to do any useful work.

Taken together, a coordinate and an SRID define a location on the
globe.  Without an SRID, a coordinate is just an abstract notion. A
“Cartesian” coordinate plane is defined as a “flat” coordinate system
placed on the surface of Earth. Because PostGIS functions work on such
a plane, comparison operations require that both geometries be
represented in the same SRID.

If you feed in geometries with differing SRIDs you will just get an
error:

```sql
SELECT ST_Equals(
        ST_GeomFromText('POINT(0 0)', 4326),
        ST_GeomFromText('POINT(0 0)', 26918)
);
```

```
ERROR:  Operation on mixed SRID geometries
CONTEXT:  SQL function "st_equals" statement 1
```

PostGIS has a table enumerating all the projections it knows about,
that you can use to lookup the correct number:

```sql
SELECT * FROM spatial_ref_sys;
```

Using the correct SRID, you can then reproject data with
`ST_Transform(geometry, srid)`. Let's transform one of our geometries
to UTM32 WGS84 (SRID 32632):

```sql
SELECT
    name,
    ST_AsText(geom) AS wgs84,
    ST_AsText(ST_Transform(geom,32632)) AS utm32
FROM
    geometries
WHERE
    name = 'Point';
```

The above example is a bit contrived, but you get the point.

Our spatial extension is intended for data analysis, thus it also
sports many function for computing things out of a geometry:

``` sql
SELECT name, ST_AsText(geom), ST_NPoints(geom), ST_Length(geom), ST_Perimeter(geom), ST_Area(geom)
FROM geometries;
```


Other functions can be used to test of compute relations between
geometries, like `ST_Equals` we have seen before. Probably the most
used one will be `ST_Distance`:

```sql
 SELECT 
  ST_Distance(
     ST_SetSRID(ST_MakePoint(-80.238,26.084), 4326),
     ST_SetSRID(ST_MakePoint(-82.355,29.644), 4326)
  ) AS distance;
```

```
      distance      
--------------------
 4.1418943733514
(1 row)
```

As you can see, the result is given in the original unit, decimal
degrees. If you want to compute the distance in kilometers, you could
do it in Euclidean space, by projecting both point to the correct
coordinate system:

```sql
--Transform from WGS 1984 to UTM Zone 17N (NAD 83)
SELECT
 ST_Distance(
   ST_Transform(ST_SetSRID(ST_MakePoint(-80.238,26.084), 4326),26917),
   ST_Transform(ST_SetSRID(ST_MakePoint(-82.355,29.644), 4326),26917)
)/1000 AS distance;
```

```
     distance     
------------------
 446.030122288446
(1 row)
```

PostGIS can be more accurate than that, though: at the cost of some
more complex calculations, you can ask it to compute the actual
distance on the WGS84 spheroid surface:

```sql
--No transformation, calculated distance on the WGS 1984 spheroid
SELECT
 ST_Distance(
   ST_SetSRID(ST_MakePoint(-80.238,26.084), 4326),
   ST_SetSRID(ST_MakePoint(-82.355,29.644), 4326),
   true
)/1000 AS distance;
```

```
    distance    
----------------
 446.18470782638
(1 row)
```

The available functions are many, many more: look them up in the
[reference](http://postgis.net/docs/manual-2.0/reference.html) to get
a grasp of what kind of tools PostGIS will offer you.

## <a name="c_1.17"></a>1.17 Visualize the coordinates of a spatial object
PostgreSQL/PostGIS offers no tool for spatial data visualization, but this can be done by a number of client applications, in particular GIS desktop software like **[ESRI ArcGIS 10.x](http://www.esri.com/software/arcgis)** or **[QGIS](http://www.qgis.org/)**. QGIS is a powerful and complete open source software. It offers all the functions needed to deal with spatial data. QGIS is the suggested GIS interface because it has many tools specifically for managing and visualizing PostGIS data. Especially remarkable is the tool *DB Manager*. Now you can explore the GPS positions data set in QGIS.

You can also use ArcGIS ESRI 10.x to visualize (but not natively edit, at least at the time of writing this text) your spatial data. Data can be accessed using “Query layers”. A query layer is a layer or stand-alone table that is defined by a SQL query. Query layers allow both spatial and non-spatial information stored in a (spatial) DBMS to be integrated into GIS projects within ArcMap. When working in ArcMap, you create query layers by defining a SQL query. The query is then run against the tables and views in a database, and the result set is added to ArcMap. Query layers behave like any other feature layer or stand-alone table, so they can be used to display data, used as input into a geoprocessing tool, or accessed using developer APIs. The query is executed every time the layer is displayed or used in ArcMap. This allows the latest information to be visible without making a copy or snapshot of the data and is especially useful when working with dynamic information that is frequently changing.

> add info on connection from material eurodeer

## <a name="c_1.18"></a>1.18 Reference systems and projections 
## <a name="c_1.19"></a>1.19 Create a point from coordinates
## <a name="c_1.20"></a>1.20 Create a line from ordered points (trajectory)
## <a name="c_1.21"></a>1.21 Calculate length of a trajectory
## <a name="c_1.22"></a>1.22 Create a polygon from points (convex hull)
## <a name="c_1.23"></a>1.23 Calculate the area of a polygon
## <a name="c_1.24"></a>1.24 Visualize spatial data in QGIS
