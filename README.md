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
```bash
shp2pgsql -I -D IDN_Road/IDN_roads.shp jalan_raya | psql --dbname=academy --username=onedata --host=10.54.4.6 --password
```

It would be upload `IDN_Roads.shp`  (`-D` dump) into table jalan_raya in database academy on server 10.54.4.6 and Will create Spatial Index (`-I`) 

### Check The data uploaded into PostgreSQL	
```sql
select gid, rtt_descri, f_code_des, geom from jalan_raya jr ;
```
And in dBeaver we can show spatial data in Map, by click tab `Spatial`

![](README/8A944B37-C682-4CD7-8E91-792CA80EA295.png)

## GEOMETRIES (Spatial Type)
### Points
“Point” or “MultiPoint”, representing one or more 0-dimensional locations. BTS (Tower), Lacci, Places, might all use a “Point” geometry type.

*In the Sandbox already table site_id with name site_id_national

#### Create POINT from table site with longitude and latitude columns
```sql
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
```sql
SELECT site_id, longitude, latitude, ST_AsText(geom)
FROM site_id_national
WHERE site_id = 'PDG001';
```


#### Extract Point data type to get longitude and latitude
```sql
SELECT site_id, longitude, latitude, ST_X(geom) as x, ST_Y(geom) as y
FROM site_id_national
WHERE site_id = 'PDG001';
```

### LineString and MultiLineString
“LineString” or “MultiLineString”, representing one or more 1-dimensional objects. Streets, streams, bus routes, power lines, driven routes, highways, might all use a “LineString” geometry type.

*previously we upload IDN_Road into database, it consist of LineString Geometry

#### Select LineString from table `jalan_raya` 
```sql
SELECT geom FROM jalan_raya where gid between 0 and 100;
```
And click spatial tab

#### Show Text format of LineStrings types from table `jalan_raya`
```sql
SELECT gid, rtt_descri, f_code_des, ST_AsText(geom) 
FROM jalan_raya 
WHERE gid < 10;
```

#### Calculate Length of LineStrings from table `jalan_raya`
```sql
SELECT gid, rtt_descri, f_code_des, 
ST_LENGTH(geom) as deg, ST_LENGTH(geom::geography) as meters
FROM jalan_raya 
WHERE gid < 10;
```

### Polygons and MultiPolygons
“Polygon” or “MultiPolygon”, representing one or more 2-dimensional objects. Census areas, parcels, boundaries, countries, neighborhoods, zoning areas, watersheds, and more.

*In the Sandbox already table border kecamatan with name `batas_kecamatan`

#### Select Polygon from table `batas_kecamatan` 
```sql
select geom from batas_kecamatan bk where region = 'SUMBAGUT';
```
![](README/4E81640A-6D65-453D-BC06-D3255B138DBE.png)

#### Show Text format of Polygons types from table `batas_kecamatan`
```sql
select region, st_astext(geom) from batas_kecamatan bk limit 1;
```

#### Calculate Area of polygons from table `batas_kecamatan`
```sql
select region, id_kec, kecamatan, st_area(geom) as deg, st_area(geom::geography)/1000 as km 
from batas_kecamatan;
```

#### Calculate Perimeter length of polygons from table `batas_kecamatan`

```sql
select region, id_kec, kecamatan, st_perimeter(geom) as deg, st_perimeter(geom::geography)/1000 as km from batas_kecamatan;
```


## SPATIAL JOIN
### ST_Intersects(A, B) and ST_Disjoint(A, B)
Intersects and disjoint are opposites. Any kind of interactions between two shapes is an intersection, and implies the pair are not disjoint, and vice versa.

#### Check Which Site Intersect with kecamatan Sukaraja in Eastern Jabotabek Region
```sql
select bk.id_kab, bk.kecamatan, site.site_id, site.longitude, site.latitude, site.geom
from batas_kecamatan bk, site_id_national site
where ST_Intersects(bk.geom, site.geom) and bk.kecamatan = 'SUKARAJA' and bk.region = 'EASTERN JABOTABEK';
```

#### Count how many Primary Roads crossing Region SUMBAGUT
```sql
select count(jln.gid) as jumlah_jalan
from jalan_raya jln, batas_kecamatan bk 
where ST_Intersects(jln.geom, bk.geom) and bk.region = 'SUMBAGUT' and jln.rtt_descri = 'Primary Route';
```

#### Check which site id JKTXXX which not in region Central Jabotabek
```sql
select site.site_id  
from (select * from batas_kecamatan where region = 'CENTRAL JABOTABEK') bk, 
(select * from site_id_national where site_id like 'JKT%') site
where ST_Disjoint(bk.geom, site.geom);
```

### ST_DWhitin(A, B, distance) and ST_Contains(A, B)
Within and contains are about objects being fully inside. One important caveat, for both functions an object on the boundary is not considered within. So a point on the outer ring of a polygon is not within the polygon.

Dwithin is object within distance of boundary object

#### Check Which `site` contains in kecamatan ‘Tangerang’ in table `batas_kecamatan`
```sql
select site.site_id, bk.kecamatan 
from batas_kecamatan bk, site_id_national site
where ST_Contains(bk.geom, site.geom);

select site.site_id, bk.kecamatan 
from batas_kecamatan bk, site_id_national site
where ST_Within(site.geom, bk.geom);
```

### ST_Distance(A, B)
Returns the shortest distance between the two geometries, in this case the distance from the point to the line mid-point.

#### Calculate Distance between each site prefix BDG
```sql
select site.site_id, site2.site_id, ST_Distance(site.geom, site2.geom) as jarak_rad,
ST_Distance(site.geom::geography, site2.geom::geography) as jarak_meter
from (select * from site_id_national where site_id LIKE 'BDG%') site, 
(select * from site_id_national where site_id LIKE 'BDG%') site2;
```

#### Calculate Distance between site and Jalan Raya
```sql
select site.site_id, jln.gid, ST_Distance(site.geom, jln.geom) as jarak_rad,
ST_Distance(site.geom::geography, jln.geom::geography) as jarak_meter
from site_id_national site, jalan_raya jln
where site_id like 'BDG%' and jln.rtt_descri = 'Primary Route'; 
```

### ST_Crosses(A, B)
Mostly used to test linestrings, which can be said to cross when their interiors have interactions. 

When linestrings cross polygon boundaries, the crosses condition is also true.

#### Check Which `jalan_raya` crosses with kecamatan `SIANTAR BARAT` 
```sql
select jln.gid, bk.kecamatan, ST_LENGTH(jln.geom::geography) as panjang_jalan, bk.geom, jln.geom
from jalan_raya jln, batas_kecamatan bk 
where ST_Crosses(jln.geom, bk.geom) and bk.kecamatan = 'SIANTAR BARAT';
```

## Clustering
### ST_VoronoiPolygons(A)
 [A Voronoi diagram](https://en.wikipedia.org/wiki/Voronoi_diagram)  is a partitioning of a plane into regions based on distance to a set of points. Each region represents the area of the plane that is closest to a particular point.

```sql
select (ST_DUMP(ST_VoronoiPolygons(ST_Collect(geom)))).geom 
from site_id_national site
where site_id like 'JKT%';
```

### ST_ClusterKMeans(A)
 [K-means clustering](https://en.wikipedia.org/wiki/K-means_clustering)  is an algorithm that divides a set of points into a specified number of clusters based on the mean distance of points from the centroid (mean) of each cluster.
The ST_ClusterKMeans function in PostGIS takes a single parameter: the number of clusters to create

```sql
SELECT kmean, count(*), ST_SetSRID(ST_Extent(geom), 4326) as bbox 
FROM
(
    SELECT ST_ClusterKMeans(geom, 20) OVER() AS kmean, ST_Centroid(geom) as geom
    FROM site_id_national 
) tsub
GROUP BY kmean;
```

### ST_ClusterDBScan
 [DBSCAN](https://en.wikipedia.org/wiki/DBSCAN)  (Density-Based Spatial Clustering of Applications with Noise) is an algorithm that clusters points based on density. Points in a high-density region are considered to be part of the same cluster, while points in a low-density region are considered to be noise (i.e., not part of any cluster).
The ST_ClusterDBSCAN function in PostGIS takes two parameters: eps and minpoints. eps is the maximum distance that points in a cluster can be from each other, and minpoints is the minimum number of points required to form a cluster.

```sql
SELECT cluster_id, count(*), ST_SetSRID(ST_Extent(geom), 4326) as bbox
from
(
SELECT  *, ST_ClusterDBSCAN(geom, eps := 0.1, minpoints := 100) over () AS cluster_id
FROM site_id_national sin2 
) bb
group by cluster_id;
```