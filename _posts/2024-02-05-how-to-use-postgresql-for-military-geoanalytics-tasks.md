---
layout: post
title: How to use PostgreSQL for (military) geoanalytics tasks 
---

I am [Taras Kloba](https://www.linkedin.com/in/kloba/), Associate
Director, Big Data & Analytics at SoftServe. I am also a co-founder of
the [PostgreSQL Ukraine
Community](https://www.facebook.com/groups/postgresql.ua/) and the
military-tech volunteer [Corvus Intelligence
project](https://corvusintell.com/), which leverages technology to
enhance our country\'s defense capabilities, particularly in
intelligence operations.

Our team won the National Defense Hackathon organized by the National
Security and Defense Council of Ukraine, and we also tackled one of the
[NATO TIDE Hackathon 2023](https://dou.ua/forums/topic/44001/)
challenges in Warsaw. My involvement in various projects and initiatives
has allowed me to gain practical experience in geoanalytics, which I am
to share in this publication.

Geoanalytics is crucial in military affairs, as a significant portion of
military data contains geoattributes. In this article, I will discuss
how to use PostgreSQL to process geospatial data and address common
geoanalytical tasks. The information will cover methods for finding the
nearest objects, distance calculations, and using geospatial indexes to
enhance these processes. We will also explore techniques for determining
a point within a polygon and geospatial aggregation. The goal of this
article is to provide practical examples and tips to enhance working
with geospatial data and contribute to the development of new solutions.

*The materials and data used in the article are open-source and have
been approved by the military representatives.*

## First data source: how to import russian military polygon data into PostgreSQL {#first-data-source-how-to-import-russian-military-polygon-data-into-postgresql .list-paragraph}

I will need certain datasets to initiate the analysis and showcase
PostgreSQL\'s capabilities in geoanalytics. I decided to start with data
on russian military facilities available on
[OpenStreetMap](https://www.openstreetmap.org/) (OSM). The first step is
to load this data into PostgreSQL, after which we can use tools to
optimize queries and enhance their efficiency.

To import data on russian military objects from OSM, we will use the
[osm2pgsql](https://osm2pgsql.org/) tool. This open-source tool
efficiently transfers data from OSM to PostgreSQL. We will load the
[russia-latest.osm.pbf](http://download.geofabrik.de/russia-latest.osm.pbf)
file (3.4 GB) containing information about points, lines, roads, and
polygons from OSM. After loading, the file will be used to populate the
corresponding tables in PostgreSQL, where we can begin the analysis and
processing of data.

The script we are using includes commands for loading OSM data, creating
a new PostgreSQL database, and importing data using osm2pgsql:

<script src="https://gist.github.com/kloba/5df9d0e76adabeda278c6d5c9cef7828.js"></script>

After executing the script, five main tables will appear in our
database:

-   **osm2pgsql_properties**---stores settings and properties used
    during the data import.

-   **planet_osm_line**---contains linear elements, such as roads and
    rivers.

-   **planet_osm_point**---includes point objects, such as buildings
    (not all buildings are marked as geographic polygons, so we will
    have to come up with something to be devised to work with these
    points).

-   **planet_osm_polygon**---stores polygons representing areas, such as
    military bases.

-   **planet_osm_roads**---stores transportation routes.

To simplify the analysis of military objects, we will create a table
called **military_geometries**. The SQL script will select data from the
**planet_osm_line**, **planet_osm_point**, **planet_osm_polygon**, and
**planet_osm_roads** tables, filtering out military objects. A 100-meter
buffer will be applied to lines, points, and roads using
[ST_Buffer](https://postgis.net/docs/ST_Buffer.html). This will also
allow us to create polygons based on points and lines, providing the
ability to analyze, for example, whether a point is within the specified
polygons.

DROP TABLE IF EXISTS military_geometries;

 

CREATE TABLE military_geometries AS

SELECT osm_id, \'line\' AS geom_type, landuse, military, building, name, operator,

       ST_Buffer(way, 0.0009)::geometry(Polygon, 4326) AS geom

FROM public.planet_osm_line

WHERE military IS NOT NULL OR building = \'military\' OR landuse = \'military\'

 

UNION ALL

 

SELECT osm_id, \'point\' AS geom_type, landuse, military, building, name, operator,

       ST_Buffer(way, 0.0009)::geometry(Polygon, 4326) AS geom

FROM public.planet_osm_point

WHERE military IS NOT NULL OR building = \'military\' OR landuse = \'military\'

 

UNION ALL

 

SELECT osm_id, \'polygon\' AS geom_type, landuse, military, building, name, operator,

       way::geometry(Polygon, 4326) AS geom

FROM public.planet_osm_polygon

WHERE military IS NOT NULL OR building = \'military\' OR landuse = \'military\'

 

UNION ALL

 

SELECT osm_id, \'road\' AS geom_type, landuse, military, building, name, operator,

       ST_Buffer(way, 0.0009)::geometry(Polygon, 4326) AS geom

FROM public.planet_osm_roads

WHERE military IS NOT NULL OR building = \'military\' OR landuse = \'military\';

 

*\--*

SELECT 9252 Query returned successfully in 12 secs 151 ms. 

Executing the provided SQL script will allow us to create a
**military_geometries** table that will contain polygons for 9,252
military objects identified on OSM:

![](vertopal_a1e72a670f354d72936e84f733154302/media/image1.png){width="6.487048337707787in"
height="4.524868766404199in"}

Visualization of 9,252 military sites across russia and the temporarily
occupied Autonomous Republic of Crimea using QGIS

In OSM, as in other open sources, information is subject to change. For
example, from the beginning of 2022, 2,995 military objects in russia
were deleted.

They say, \"Screenshots don\'t burn\", but such deletions often lead to
the [Streisand effect](https://en.wikipedia.org/wiki/Streisand_effect),
where attempts to hide information only attract more attention. If you
want to delve into the historical data of OSM and help identify such
anomalies, you can use resources like
[GeoFabrik.de](https://download.geofabrik.de/russia.html). Although this
doesn\'t directly relate to our analysis, I want to show how these
deleted objects look on a map, illustrating russian attempts to conceal
essential data.

![](vertopal_a1e72a670f354d72936e84f733154302/media/image2.jpeg){width="6.875in"
height="5.922916666666667in"}

Deleted after 01/01/2022 (blue) and existing (red) geographical polygons
of military facilities in moscow

## Second data source: fire data from NASA satellites {#second-data-source-fire-data-from-nasa-satellites .list-paragraph}

As the next data source, we will utilize information from the [Fire
Information for Resource Management
System](https://www.earthdata.nasa.gov/learn/find-data/near-real-time/firms/vj114imgtdlnrt)
(FIRMS) developed at the University of Maryland with support from NASA
and the UN in 2007. FIRMS allows real-time monitoring of active fires
worldwide, utilizing data from Aqua and Terra satellites equipped with
MODIS spectroradiometers and VIIRS on S-NPP and NOAA 20 satellites. The
information is updated every three hours and even more frequently for
the United States and Canada.

We will be using FIRMS data to identify fires within the territory of
russian military facilities since 2022.

To download fire data from the FIRMS system, we will employ the
following script, extracting all records of fires in russia from January
1, 2022, to the current date. These data will then be imported into a
new table, **viirs_fire_events**, in the PostgreSQL database.

from datetime import datetime, timedelta

import time

import requests

import psycopg2

import csv

from psycopg2.extras import execute_values

 

*# Your NASA FIRMS API key*

api_key = \'your_api_key\'

 

*# Start and end dates for the data retrieval*

start_date = datetime(2022, 1, 1)

end_date = datetime(2023, 12, 24)

 

*# URL for the NASA FIRMS API call*

base_url = \"https://firms.modaps.eosdis.nasa.gov/api/country/csv/{}/VIIRS_NOAA20_NRT/RUS/1/{}\"

 

*# Database connection details*

db_name = \'your_db_name\'

db_user = \'your_db_user\'

db_password = \'your_db_password\'

db_host = \'your_db_host\'

db_port = \'your_db_port\'

 

*# Function to insert data into PostgreSQL*

def insert_data_into_db(data, cursor):

    insert_query = \"\"\"INSERT INTO viirs_fire_events (country_id, latitude, longitude, bright_ti4, scan, track, acq_date, acq_time, satellite, instrument, confidence, version, bright_ti5, frp, daynight) VALUES %s;\"\"\"

    execute_values(cursor, insert_query, data)

 

*# Function to fetch and insert data into the database*

def fetch_and_insert_data(date, cursor):

    formatted_date = date.strftime(\"%Y-%m-%d\")

    try:

        response = requests.get(base_url.format(api_key, formatted_date))

        

        if \"Invalid MAP_KEY\" in response.text:

            print(f\"Invalid MAP_KEY detected for {formatted_date}. Waiting for 10 minutes before retrying.\")

            time.sleep(600)  *# Wait for 10 minutes*

            return fetch_and_insert_data(date, cursor)  *# Retry*

 

        decoded_content = response.content.decode(\'utf-8\')

        csv_reader = csv.reader(decoded_content.splitlines(), delimiter=\',\')

        next(csv_reader, None)  *# Skip the header*

        rows = \[tuple(row) for row in csv_reader if row\]

 

        if rows:

            insert_data_into_db(rows, cursor)

            print(f\"Data for {formatted_date} inserted successfully (rows = {len(rows)}).\")

        else:

            print(f\"No valid data to insert for {formatted_date}.\")

 

    except Exception as e:

        *# General exception handler for any other unexpected errors*

        print(f\"Error fetching data for {formatted_date}: {e}\")

 

try:

    conn = psycopg2.connect(dbname=db_name, user=db_user, password=db_password, host=db_host, port=db_port)

    cur = conn.cursor()

 

    current_date = start_date

    while current_date \<= end_date:

        fetch_and_insert_data(current_date, cur)

        conn.commit()

        current_date += timedelta(days=1)

    

except Exception as e:

    print(f\"Database error: {e}\")

finally:

    if cur is not None:

        cur.close()

    if conn is not None:

        conn.close()

 

print(\"Data fetching and insertion complete.\")

Therefore, we will populate the **viirs_fire_events** table, which will
contain 1,711,475 records of fires in russia. These fires appear as
follows:

![](vertopal_a1e72a670f354d72936e84f733154302/media/image3.jpeg){width="6.881944444444445in"
height="3.3541666666666665in"}

Visualization of fires in russia since January 1, 2022 (1,711,475 fires)

The **viirs_fire_events** table in the PostgreSQL database will be used
to store detailed fire data, with fields for coordinates, satellite
parameters, date and time of acquisition, and other critical metadata. A
new column with a data type of **GEOMETRY(POINT, 4326)** will be
automatically populated based on the data from the **longitude** and
**latitude** columns.

CREATE TABLE viirs_fire_events (

    country_id TEXT,

    latitude FLOAT,

    longitude FLOAT,

    bright_ti4 FLOAT,

    scan FLOAT,

    track FLOAT,

    acq_date DATE,

    acq_time INT,

    satellite TEXT,

    instrument TEXT,

    confidence TEXT,

    version TEXT,

    bright_ti5 FLOAT,

    frp FLOAT,

    daynight TEXT,

    geom GEOMETRY(POINT, 4326) GENERATED ALWAYS AS (ST_MakePoint(longitude, latitude)) STORED

); 

Suppose you are interested in working with this data on military objects
and fires, but the described process of extracting datasets seems
time-consuming. In that case, there are exported CSV tables for you. You
can download them via the following links:
[military_geometries](https://storage.googleapis.com/files.sql.ua/csv/military_geometries.csv),
[viirs_fire_events](https://storage.googleapis.com/files.sql.ua/csv/viirs_fire_events.csv).

## Searching for military facilities where fires occurred: points within the polygon {#searching-for-military-facilities-where-fires-occurred-points-within-the-polygon .list-paragraph}

As for now, we have two tables: **military_geometries** and
**viirs_fire_events**. Let\'s try to find those military facilities that
have had fires (since the beginning of 2022) or those that have not yet
🙂.

![](vertopal_a1e72a670f354d72936e84f733154302/media/image4.png){width="2.1411439195100614in"
height="7.033333333333333in"}

Let\'s use an SQL query with the
[**ST_Contains**](https://postgis.net/docs/ST_Contains.html) function to
identify military objects where fires have been detected from NASA
satellites.

SELECT mg.osm_id, 

    mg.geom_type, 

    mg.landuse, 

    mg.military, 

    mg.building, 

    mg.name, 

    mg.operator, 

    mg.geom

FROM public.military_geometries AS mg

WHERE EXISTS (

    SELECT 1

    FROM public.viirs_fire_events AS vfe

    WHERE ST_Contains(mg.geom, vfe.geom)

);

*\--*

Successfully run. Total query runtime: 54 min 15 secs. 129 rows affected. 

As you\'ve probably noticed, we\'ve identified 129 military sites that
have experienced fires since the start of 2022. What\'s intriguing is
that, in some cases, these fires seem to have occurred more than once.

![](vertopal_a1e72a670f354d72936e84f733154302/media/image5.png){width="7.013320209973753in"
height="2.8860126859142605in"}

Military facilities where fires have occurred since the beginning of
2022 (the transparency of the facilities indicates the frequency of the
fire incidents)

The second aspect you may have noticed is that the specified query took
54 minutes and 15 seconds to execute, which is quite long for such a
straightforward operation. It\'s helpful to use the [EXPLAIN
ANALYZE](https://www.postgresql.org/docs/current/sql-explain.html)
command to understand the reasons for this duration. This command allows
you to analyze the query execution process, identify potential
bottlenecks, and further optimize the query to improve performance.

Nested Loop Semi Join  (cost=0.00..395827843592.52 rows=8 width=524) (actual time=4628.471..4005790.130rows=129 loops=1)

  Join Filter: st_contains(mg.geom, vfe.geom)

  Rows Removed by Join Filter: 15695214562

  -\>  Seq Scan on military_geometries mg  (cost=0.00..649.52 rows=9252 width=524) (actual time=0.009..22.516rows=9252 loops=1)

  -\>  Materialize  (cost=0.00..70285.12 rows=1711475 width=32) (actual time=0.015..121.310 rows=1696413loops=9252)

        -\>  Seq Scan on viirs_fire_events vfe  (cost=0.00..50027.75 rows=1711475 width=32) (actual time=124.339..351.922 rows=1711475 loops=1)

Planning Time: 0.923 ms

Execution Time: 4005798.854 ms 

In this case, when using the **Nested Loop Semi Join** operator, we
encountered a complexity of O(n\*m), where n is 9,252 rows in the
**military_geometries** table, and m is 1,711,475 rows in
**viirs_fire_events**. This implies that each row from the first table
is compared with every row from the second table, resulting in a huge
number of operations.

Hence, let\'s discuss how we can speed up the execution of such a query
by utilizing indexes.

## Productivity boost: utilizing indexing in geoanalytics {#productivity-boost-utilizing-indexing-in-geoanalytics .list-paragraph}

PostgreSQL is renowned for its scalability features, offering numerous
methods for accessing geospatial data within this database. To find all
methods suitable for working with points in a two-dimensional space, we
can execute the following query:

SELECT am.amname AS access_method, typ.typname, opc.opcname AS operator_class 

FROM pg_am am

INNER JOIN pg_opclass opc ON am.oid = opc.opcmethod

INNER JOIN pg_type typ ON typ.oid = opc.opcintype

WHERE (opc.opcname LIKE \'%geometry%\' OR opc.opcname LIKE \'%point%\')

AND (

    opc.opcname NOT LIKE \'%3d%\' AND

    opc.opcname NOT LIKE \'%4d%\' AND

    opc.opcname NOT LIKE \'%nd%\'

); 

As a result, we will observe at least five access methods, including
btree, hash, gist, brin, and spgist. I suggest investigating by creating
indexes for each of these methods and operator classes. After creating
the indexes, we will assess the query performance regarding fires at
military facilities in russia to determine which methods are most
effective for our task.

  -----------------------------------------------------------------------------------------------------------------------
  Index type    Index operator class             Filtering operator     Index      Index   Query       Brief explanation
                                                                        creation   size    execution   
                                                                        time               time        
  ------------- -------------------------------- ---------------------- ---------- ------- ----------- ------------------
  btree         btree_geometry_ops               There is no            1 sec 918  81 MB   53 min 45   Supports equality
                                                 corresponding          ms                 sec (129    and range queries;
                                                 operator---the index                      rows        retrieves data
                                                 is disregarded for                        affected)   quickly and in an
                                                 this query.                                           organized manner.

  hash          hash_geometry_ops                There is no            3 secs 158 59 MB   53 min 15   Fast equality
                                                 corresponding          ms                 sec (129    search; not
                                                 operator---the index                      rows        suitable for
                                                 is disregarded for                        affected)   ordering or range
                                                 this query.                                           queries.

  brin          brin_geometry_inclusion_ops_2d   @(geometry,geometry)   536 ms     0.032   28 min 3    Effective for
                                                                                   MB      sec (129    large datasets
                                                                                           rows        with naturally
                                                                                           affected)   ordered data;
                                                                                                       indexes block
                                                                                                       ranges rather than
                                                                                                       individual rows.

  gist          gist_geometry_ops_2d             @(geometry,geometry)   11 secs    94 MB   493 ms (129 Supports a wide
                                                                        659 ms             rows        range of queries,
                                                                                           affected)   including spatial
                                                                                                       searches for
                                                                                                       overlap and
                                                                                                       proximity.

  spgist        spgist_geometry_ops_2d           @(geometry,geometry)   6 secs 290 78 MB   353 ms (129 Suitable for data
                                                                        ms                 rows        with uneven
                                                                                           affected)   distribution;
                                                                                                       supports a variety
                                                                                                       of split tree
                                                                                                       structures.

  The following                                                                                        
  operator                                                                                             
  classes do                                                                                           
  not utilize                                                                                          
  geometry data                                                                                        
  types for                                                                                            
  searching;                                                                                           
  instead, they                                                                                        
  work with                                                                                            
  \<@(point,                                                                                           
  polygon) and                                                                                         
  \<@(point,                                                                                           
  box). As a                                                                                           
  result, the                                                                                          
  row counts                                                                                           
  may not match                                                                                        
  the output                                                                                           
  (for example,                                                                                        
  a complex                                                                                            
  geographic                                                                                           
  polygon may                                                                                          
  have been                                                                                            
  simplified to                                                                                        
  a rectangle).                                                                                        

  gist          point_ops                        \<@(point,polygon)     1 secs 426 81 MB   306 ms      Perfect for point
                                                                        ms                 (returned   data; supports
                                                                                           132         queries on spatial
                                                                                           records)    relationships,
                                                                                                       such as
                                                                                                       containment and
                                                                                                       intersection.

  spgist        quad_point_ops                   \<@(point,box)         4 secs 849 77 MB   243 ms (173 Utilizes quadtrees
                                                                        ms                 rows        to index point
                                                                                           affected)   data; effective in
                                                                                                       specific scenarios
                                                                                                       of spatial
                                                                                                       analysis.

  spgist        kd_point_ops                     \<@(point,box)         5 secs 204 93 MB   199 ms (173 Employs kd-trees
                                                                        ms                 rows        for
                                                                                           affected)   multidimensional
                                                                                                       point data;
                                                                                                       excellent for
                                                                                                       finding nearest
                                                                                                       neighbors.
  -----------------------------------------------------------------------------------------------------------------------

The results table shows that the most effective indexes for our task are
[GiST](https://www.postgresql.org/docs/current/gist.html) and
[SP-GiST](https://www.postgresql.org/docs/current/spgist.html). Let\'s
delve into how they operate.

## How GiST works {#how-gist-works .list-paragraph}

Generalized Search Tree (GiST) indexes in PostgreSQL enable efficient
sorting and searching across diverse data types using the concept of
balanced trees. They provide the ability to develop custom operators for
indexing, making GiST quite versatile and adaptive to specific
requirements.

![](vertopal_a1e72a670f354d72936e84f733154302/media/image6.png){width="6.9in"
height="5.945138888888889in"}

The hierarchical structure of the GiST index in PostgreSQL \[1\]

In the example of a GiST tree depicted: at the top level, there are
**R1** and **R2**, serving as bounding boxes for other elements. **R1**
contains **R3**, **R4**, and **R5**, while **R3**, in turn, encompasses
**R8**, **R9**, and **R10**. The GiST index has a hierarchical
structure, allowing for significantly faster search. Unlike B-trees,
GiST supports overlap operations and spatial relationship determination.
This is why GiST is well-suited for indexing geometric data.

## How SP-GiST works {#how-sp-gist-works .list-paragraph}

Space Partitioning Generalized Search Tree (SP-GiST) indexes in
PostgreSQL are designed for data structures that partition space into
non-overlapping regions, such as quadrant trees or prefix trees. They
enable the recursive division of data into subsets, forming unbalanced
trees. This makes SP-GiST indexes particularly effective for in-memory
usage, where they can quickly process queries due to fewer levels and
small data groups in each node.

However, SP-GiST indexes have disadvantages when stored on disk due to
the high number of disk operations required for their functioning,
especially in large databases.

Considering this, GiST indexes often become a better choice, especially
when working with polygons and complex spatial structures.

## Finding the nearest neighbors: 10 fires near the Shahed production plant {#finding-the-nearest-neighbors-10-fires-near-the-shahed-production-plant .list-paragraph}

Now, let\'s attempt to solve the task of finding nearest neighbors using
PostgreSQL. Using our datasets, we will try to identify ten fires that
occurred near the factory in russia, where Iranian Shahed drones are
manufactured. For more detailed information about the plant, you can
refer to the [research](https://molfar.com/blog/alabuga-deanon)
conducted by the Molfar team. The factory is located in the special
economic zone
[Alabuga](https://en.wikipedia.org/wiki/Alabuga_Special_Economic_Zone)
in Tatarstan, where previously cat food and automotive glass were
produced, and mushrooms were grown. However, after sanctions against
Russia, its priorities shifted, and now it plays a key role in russia\'s
plans for drone production.

One of the methods to solve this task is to create a buffer in the form
of a circle around the selected target. This buffer is recursively
expanded until the required number of results is obtained. In
PostgreSQL, this can be implemented with the following SQL query, which
forms the buffer and identifies fires that occurred within the specified
radius from the selected object:

SELECT \*

FROM public.viirs_fire_events

WHERE ST_DWithin(

    geom, 

    ST_SetSRID(ST_MakePoint(52.048470, 55.821688), 4326)::geography, 

    10000 *\-- can be changed to 1000, 5000, 10000, \...*

)

LIMIT 10;

 

*\--*

Successfully run. Total query runtime: 1 secs 157 ms.

10 rows affected. 

This approach involves gradually expanding the buffer and analyzing the
results, which can be time-consuming.

![](vertopal_a1e72a670f354d72936e84f733154302/media/image7.png){width="6.901715879265092in"
height="4.993055555555555in"}

A plant in Tatarstan that produces Shaheds with fire visualization
within radii of 1.5 and 10 km

Various operators supported by GiST indexes can be utilized to optimize
geospatial queries. To retrieve a list of available operators to use
with a GiST index, you can execute an SQL query that scans the
PostgreSQL system tables and provides information about operators
associated with the **gist_geometry_ops_2d** operator class. This will
help identify the most efficient operators for performing specific
geospatial operations in the database.

SELECT

    amopopr::regoperator, 

    oprcode::regproc, 

    obj_description(opr.oid, \'pg_operator\') description

FROM pg_am am

INNER JOIN pg_opclass opc ON opcmethod = am.oid

INNER JOIN pg_amop amop ON amopfamily = opcfamily

INNER JOIN pg_operator opr ON opr.oid = amopopr

WHERE amname = \'gist\'

    AND opcname = \'gist_geometry_ops_2d\'

ORDER BY amopstrategy;

 

*\--*

 amopopr                \|    oprcode            \| description

*\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--*

 \<\<(geometry,geometry)  \| geometry_left         \| Determines if one geometry is left of another

 &\<(geometry,geometry)  \| geometry_overleft     \| Checks if one geometry overlaps the left of another

 &&(geometry,geometry)  \| geometry_overlaps     \| Checks if two geometries overlap

 &\>(geometry,geometry)  \| geometry_overright    \| Determines if one geometry overlaps the right of another

 \>\>(geometry,geometry)  \| geometry_right        \| Determines if one geometry is right of another

 \~=(geometry,geometry)  \| geometry_same         \| Checks if two geometries are the same

 \~(geometry,geometry)   \| geometry_contains     \| Determines if one geometry contains another

 @(geometry,geometry)   \| geometry_within       \| Determines if one geometry is within another

 &\<\|(geometry,geometry) \| geometry_overbelow    \| Determines if one geometry overlaps below another

 \<\<\|(geometry,geometry) \| geometry_below        \| Determines if one geometry is below another

 \|\>\>(geometry,geometry) \| geometry_above        \| Determines if one geometry is above another

 \|&\>(geometry,geometry) \| geometry_overabove    \| Determines if one geometry overlaps above another

 \<-\>(geometry,geometry) \| geometry_distance_centroid \| Calculates centroid distance between geometries

 \<#\>(geometry,geometry) \| geometry_distance_box \| Calculates box distance between geometries 

Our GiST index provides extensive capabilities for working with geodata,
allowing you to determine the spatial location of objects and measure
distances. The **\<-\>** operator enables sorting objects by proximity
to a specified point. In this example, we use this operator to identify
the ten closest fires to the specified location.

SELECT \*

FROM public.viirs_fire_events

ORDER BY geom \<-\> ST_SetSRID(ST_MakePoint(52.048470, 55.821688), 4326)

LIMIT 10;

 

*\--*

Successfully run. Total query runtime: 78 ms.

10 rows affected. 

The query turned out to be significantly faster---15 times swifter,
compared to the previous methodology, and this is without repeated
executions with a changed radius. We can analyze the query plan to
confirm that the speed increased due to the use of an index and an
operator. This way, we\'ll ensure that the index was indeed involved,
which is the key to improving productivity.

EXPLAIN ANALYZE

SELECT \*

FROM public.viirs_fire_events

ORDER BY geom \<-\> ST_SetSRID(ST_MakePoint(52.048470, 55.821688), 4326)

LIMIT 10;

 

*\--*

QUERY PLAN

Limit  (cost=0.41..14.22 rows=10 width=143) (actual time=0.230..0.254 rows=10 loops=1)

  -\>  Index Scan using idx_viirs_fire_events_geom_gist on viirs_fire_events  (cost=0.41..2363046.98rows=1711475 width=143) (actual time=0.229..0.252 rows=10 loops=1)

        Order By: (geom \<-\> \'0101000020E6100000276BD44334064A4001C287122DE94B40\'::geometry)

Planning Time: 0.105 ms

Execution Time: 0.283 ms 

As we can see, indexes, similar to GiST, extend analytical capabilities
beyond simple comparisons, enabling the resolution of more complex
tasks. As demonstrated in this article, open data can be effectively
utilized for quickly assessing and defining goals on a global scale,
including evaluating the success of target impact.

## Uber\'s H3: a perspective on geospatial analytics and data aggregation {#ubers-h3-a-perspective-on-geospatial-analytics-and-data-aggregation .list-paragraph}

The H3, developed by Uber, is a hexagonal grid system designed to
facilitate flexible and efficient distribution of geospatial data. It
seems that H3 has the potential to become a common standard for working
with geodata in the Armed Forces of Ukraine. Let\'s explore how this
tool can be used for data aggregation and solving complex geoanalytical
tasks.

![](vertopal_a1e72a670f354d72936e84f733154302/media/image8.png){width="4.674759405074366in"
height="4.465277777777778in"}

Illustration of the Uber H3 hexagonal grid

As you can see in the image, each hexagon serves as a distinct
geographic unit, simplifying the processing of intricate geoforms into
uniform segments.

  ------------------------------------------------------------------------------
  Level            Total number of       Number of hexagons    Number of
                   objects                                     pentagons
  ---------------- --------------------- --------------------- -----------------
  0                122                   110                   12

  1                842                   830                   12

  2                5,882                 5,870                 12

  3                41,162                41,150                12

  4                288,122               288,110               12

  5                2,016,842             2,016,830             12

  6                14,117,882            14,117,870            12

  7                98,825,162            98,825,150            12

  8                691,776,122           691,776,110           12

  9                4,842,432,842         4,842,432,830         12

  10               33,897,029,882        33,897,029,870        12

  11               237,279,209,162       237,279,209,150       12

  12               1,660,954,464,122     1,660,954,464,110     12

  13               11,626,681,248,842    11,626,681,248,830    12

  14               81,386,768,741,882    81,386,768,741,870    12

  15               569,707,381,193,162   569,707,381,193,150   12
  ------------------------------------------------------------------------------

This is a hierarchical system consisting of 15 levels dividing the
Earth\'s surface into hexagons. The zero level is divided into 122
sections, 12 of which are pentagons for accurately representing the
Earth\'s spherical shape. We have approximately 569 trillion hexagons at
the finest level, each representing a distinct geospatial object. The
video below demonstrates how this works in practice.
<https://youtu.be/RbeYPqsFGPI>

PostgreSQL can integrate H3 functionality through an additional
extension. To install this extension, use the **CREATE EXTENSION h3;**
command, and it is available on cloud computing services, including AWS
RDS (my acknowledgments to AWS for their support of Ukraine). Once
you\'ve installed this extension, new functions become accessible.
Let\'s explore those functions that might be useful for beginners:

  -------------------------------------------------------------------------------
  Function              Input data       Output data         Description
  --------------------- ---------------- ------------------- --------------------
  h3_lat_lng_to_cell    latitude: FLOAT, H3 index: BIGINT    Converts latitude
                        longitude:                           and longitude
                        FLOAT,                               coordinates into an
                        resolution: INT                      H3 index at a
                                                             specified resolution
                                                             level.

  h3_cell_to_boundary   H3 index: BIGINT Array of boundary   Transforms an H3
                                         coordinates:        index into a
                                         GEOMETRY(POLYGON,   geometric polygon
                                         4326)               representing the
                                                             boundaries of a
                                                             hexagon.

  h3_get_resolution     H3 index: BIGINT Resolution level:   Returns the
                                         INT                 resolution level of
                                                             a given H3 index.

  h3_cell_to_parent     H3 index:        Parent H3 index:    Converts an H3 index
                        BIGINT, desired  BIGINT              into its parent
                        resolution: INT                      index at a higher
                                                             hierarchy level.

  h3_cell_to_children   H3 index:        Array of child      Converts an H3 index
                        BIGINT, desired  H3 indexes: SETOF   into an array of
                        resolution: INT  BIGINT              child indices at a
                                                             lower hierarchy
                                                             level.

  h3_polygon_to_cells   geometry:        Array               Transforms a polygon
                        GEOMETRY,        of H3 indexes:      into a set of H3
                        resolution: INT  SETOF BIGINT        indices that fully
                                                             or partially cover
                                                             the polygon.

  h3_grid_disk          H3 index:        Array               Generates an array
                        BIGINT, range:   of H3 indexes:      of H3 indices
                        INT              SETOF BIGINT        representing a
                                                             hexagonal grid
                                                             around the central
                                                             H3 index, forming a
                                                             \"disk\" of a
                                                             defined radius.

  h3_compact_cells      Array            Array of compact    Consolidates an
                        of H3 indexes:   H3 indexes: SETOF   array of H3 indices,
                        SETOF BIGINT     BIGINT              reducing the number
                                                             of indices covering
                                                             the same area.
  -------------------------------------------------------------------------------

To address the first task effectively, we can transform all our polygons
into arrays of H3 indexes (hexagons) of a specified level. Similarly, we
can process the centroids of fires by converting them into H3 indexes.
By obtaining BIGINT data types for these H3 indexes, we can apply a
standard B-tree index, which is particularly efficient in performing
equality comparison operations. This will significantly improve query
execution speed in complex geoanalysis tasks, ensuring fast and accurate
results.

Let\'s examine a few simple H3 functions that will help better
understand how this works in practice:

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  ![](vertopal_a1e72a670f354d72936e84f733154302/media/image9.png){width="2.175068897637795in"   ![](vertopal_a1e72a670f354d72936e84f733154302/media/image10.png){width="2.3154615048118985in"   ![](vertopal_a1e72a670f354d72936e84f733154302/media/image11.png){width="2.308632983377078in"
  height="2.6in"}                                                                               height="2.6in"}                                                                                 height="2.6in"}
  --------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------
  h3_polygon_to_cells(geom, 8)                                                                  h3_grid_disk(h3_polygon_to_cells(geom, 8), 1)                                                   h3_polygon_to_cells(geom, 9)

  This function converts the geometry of a military polygon into a set of H3 indexes at the     If certain areas of the polygon remain uncovered after applying h3_polygon_to_cells, you can    Using the **h3_polygon_to_cells** function with a level 9 increases the grid\'s resolution to
  eighth level of resolution, effectively dividing the polygon into hexagons, each covering an  use h3_grid_disk to create an additional ring of H3 indexes. It will expand coverage by adding  a finer scale, where each hexagon represents an area of 0.105332513 square kilometers. This
  area of 0.737327598 square kilometers, enabling detailed spatial analysis.                    hexagons around existing indexes, ensuring complete coverage of the defined geographic polygon. allows for greater accuracy in reproducing the geometry of the geographic polygon for detailed
                                                                                                                                                                                                spatial analysis. However, it also results in more hexagons, which may negatively impact query
                                                                                                                                                                                                execution speed.
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

During my presentation at PGConf.2023, the largest conference in Europe
dedicated to PostgreSQL, I had the opportunity to showcase a series of
more complex challenges that can be addressed by aggregating geospatial
data using H3. One example involved the search for other drones located
in the exact location and time, as well as the analysis of routes taken
by drones traveling together, identified in different places over a
specific period (Companion Analysis). You will have the opportunity to
learn more about this topic in the continuation of this article. In the
meantime, you can check out the materials of my
[presentation](https://klioba.com/public/presentations/PostGIS_Warfare_Export.pdf).

Within our datasets, we can analyze military objects and, through
aggregation with H3, calculate the density of these objects in russia.
The visualization of this analysis looks like this:

![](vertopal_a1e72a670f354d72936e84f733154302/media/image12.png){width="6.888888888888889in"
height="4.221579177602799in"}

Visualization of the density of military objects in russia and the
temporarily occupied Autonomous Republic of Crimea using H3 hexagons

Using H3 for aggregating geospatial data significantly enhances
analytical capabilities, allowing for a more profound interpretation and
visualization of complex spatial relationships.

Concluding remarks

If you are a representative of the Armed Forces of Ukraine and are
seeking qualified support in the field of data, Big Data, or
geoanalytics, feel free to reach out to me. My team of volunteers and I
will gladly assist you with our knowledge and resources.

Additional resources I utilized in preparing this article:

-   [Mastering PostgreSQL by Hans-Jürgen
    Schönig](https://subscription.packtpub.com/book/data/9781800567498/3/ch03lvl1sec19/understanding-postgresql-index-types)

-   [Inside the russian effort to build 6,000 attack drones with Iran's
    help](https://www.washingtonpost.com/investigations/2023/08/17/russia-iran-drone-shahed-alabuga/)

-   [Uber H3. Tables of Cell Statistics Across
    Resolutions](https://h3geo.org/docs/core-library/restable/)

-   [H3-PG Extension. API
    Reference](https://github.com/zachasme/h3-pg/blob/main/docs/api.md)

-   [PostGIS documentation](https://postgis.net/documentation/)

-   [PostGIS in Action, Third Edition by Leo S. Hsu and Regina
    Obe](https://www.amazon.com/PostGIS-Action-Third-Leo-Hsu/dp/1617296694)

While preparing this article, I used an Ubuntu server with the following
characteristics and configured the PostgreSQL database as follows:

\# DB Version: 16

\# OS Type: linux

\# DB Type: dw

\# Total Memory (RAM): 8 GB

\# CPUs num: 4

\# Connections num: 20

\# Data Storage: ssd

max_connections = 20

shared_buffers = 2 GB

effective_cache_size = 6 GB

maintenance_work_mem = 1 GB

checkpoint_completion_target = 0.9

wal_buffers = 16 MB

default_statistics_target = 500

random_page_cost = 1.1

effective_io_concurrency = 200

work_mem = 26214 kB

huge_pages = off

min_wal_size = 4 GB

max_wal_size = 16 GB

max_worker_processes = 4

max_parallel_workers_per_gather = 2

max_parallel_workers = 4

max_parallel_maintenance_workers = 2

\# Recommendations are generated by pgtune.leopard.in.ua
