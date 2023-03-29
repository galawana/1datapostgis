# 1Data Academy - Fundamental PostGIS

## INTRODUCTION
PostGIS is a spatial database extender for  [PostgreSQL](https://postgresql.org/)  object-relational database. It adds support for geographic objects allowing location queries to be run in SQL.

### What is SPATIAL Database
System for storage and random access of relationally (tables of rows and columns) structured data, providing the following capabilities for that data.
* **Data Types** including Spatial Types
* number, date, string, geometry, geography and raster
* **Indexes** including Spatial Indexes
* b-tree, hash, rtree, quadtree
* **Functions** including Spatial Functions
strlen(string), pow(float, float), now(), ST_Area(), ST_Distance() 


## LOADING Spatial Data into PostgreSQL
We need postgis tools to load Spatial Data for PostgreSQL we can use **shp2pgsql (included in PostGIS installation)**

### Load IDN_Road ShapeFile into PostgreSQL
Running the following command from terminal (Linux, Mac) 
```
shp2pgsql -I -D IDN_Road/IDN_roads.shp jalan_raya | psql --dbname=academy --username=onedata --host=10.54.4.6 --password
```

It would be upload `IDN_Roads.shp`  (`-D` dump) into table jalan_raya in database academy on server 10.54.4.6 and Will create Spatial Index (`-I`) 

### Check The data uploaded into PostgreSQL	
```
select gid, rtt_descri, f_code_des, geom from jalan_raya jr ;
```
And in dBeaver we can show spatial data in Map, by click tab `Spatial`

![](README/8A944B37-C682-4CD7-8E91-792CA80EA295.png)

## GEOMETRIES (Spatial Type)
### Points
“Point” or “MultiPoint”, representing one or more 0-dimensional locations. BTS (Tower), Lacci, Places, might all use a “Point” geometry type.

*In the Sandbox already table site_id with name site_id_national

#### Create POINT from table site with longitude and latitude columns
```
## change <IAM> with your unique name
CREATE TABLE IF NOT EXISTS site_id_<IAM> (
   site_id varchar(20) primary key,
   node varchar(100) null,
   site_name varchar(100) null,
   longitude decimal(12, 9) not null, 
   latitude decimal(12, 9) not null
);

INSERT INTO site_id_<IAM> SELECT * FROM site_id_national WHERE site_id LIKE 'PDG%';

SELECT AddGeometryColumn('site_id_national', 'geom', 4326, 'POINT', 2 );

UPDATE site_id_national 
SET geom = ST_SetSRID(ST_MakePoint(longitude, latitude), 4326);
```

#### Show Text format of Points types from given table
```
SELECT site_id, longitude, latitude, ST_AsText(geom)
FROM site_id_national
WHERE site_id = 'PDG001';
```


#### Extract Point data type to get longitude and latitude
```
SELECT site_id, longitude, latitude, ST_X(geom) as x, ST_Y(geom) as y
FROM site_id_national
WHERE site_id = 'PDG001';
```

### LineString and MultiLineString
“LineString” or “MultiLineString”, representing one or more 1-dimensional objects. Streets, streams, bus routes, power lines, driven routes, highways, might all use a “LineString” geometry type.

*previously we upload IDN_Road into database, it consist of LineString Geometry

#### Select LineString from table `jalan_raya` 
```
SELECT geom FROM jalan_raya where gid between 0 and 100;
```
And click spatial tab

#### Show Text format of LineStrings types from table `jalan_raya`
```
SELECT gid, rtt_descri, f_code_des, ST_AsText(geom) 
FROM jalan_raya 
WHERE gid < 10;
```

#### Calculate Length of LineStrings from table `jalan_raya`
```
SELECT gid, rtt_descri, f_code_des, 
ST_LENGTH(geom) as deg, ST_LENGTH(geom::geography) as meters
FROM jalan_raya 
WHERE gid < 10;
```

### Polygons and MultiPolygons
“Polygon” or “MultiPolygon”, representing one or more 2-dimensional objects. Census areas, parcels, boundaries, countries, neighborhoods, zoning areas, watersheds, and more.

*In the Sandbox already table border kecamtan with name `batas_kecamatan`

#### Select Polygon from table `batas_kecamatan` 
```
select geom from batas_kecamatan bk where region = 'SUMBAGUT';
```
![](README/4E81640A-6D65-453D-BC06-D3255B138DBE.png)

#### Show Text format of Polygons types from table `batas_kecamatan`
```
select region, st_astext(geom) from batas_kecamatan bk limit 1;
```

#### Calculate Area of polygons from table `batas_kecamatan`
```
select region, id_kec, kecamatan, st_area(geom) as deg, st_area(geom::geography)/1000 as km 
from batas_kecamatan;
```

#### Calculate Perimeter length of polygons from table `batas_kecamatan`

```
select region, id_kec, kecamatan, st_perimeter(geom) as deg, st_perimeter(geom::geography)/1000 as km from batas_kecamatan;
```


## SPATIAL JOIN
### 