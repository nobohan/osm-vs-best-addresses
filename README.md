BEST addresses in Belgium vs OSM
=================================

This small repository is about how to compare the addresses in OpenStreetMap against the official addresses from Best Address. 

The processing is made (mainly) in QGIS.


## Download data

1) Best addresses

Addresses are regularly updated and are available at https://opendata.bosa.be/index.fr.html

The single geopackage from NGI/IGN is handy, since it is just one file: https://www.geo.be/catalog/details/ca0fd5c0-8146-11e9-9012-482ae30f98d9?l=fr

2) OSM data

For OSM data, we used a classical postgresql/postgis database filled with osm2pgsql. Note that you may also use geopackage files from geofabrik, but I already have a postgis database with OSM data on my pc.

- Download a pbf file for Belgium from geofabrik
- Import in a postGIS database via osm2pgsql

3) Admin boundaries from IGN

Download the admin boundaries from NGI/IGN at https://www.geo.be/catalog/details/fb1e2993-2020-428c-9188-eb5f75e284b9?l=en. 

We'll use the municipalities layer to compute the number of addresses per municipalities. 

## Data processing

### Best addresses

- add the gpkg layer
- filter by "status" = 'current'
- remove duplicate locations: In best address, many address are duplicated at the same location for each postbox (box number) of one building number. This is not the case in OSM so we need to remove duplicate addresses. 

### OSM addresses

- Add the planet_osm_polygon
- filter planet_osm_polygon by `building IS NOT NULL AND addr:housenumber IS NOT NULL`
- centroid tool on this layer -> save as "osm_addr_ways_centroid.gpkg" (takes some times)

- Add the planet_osm_point 
- filter by `addr:housenumber IS NOT NULL`

- select by location: planet_osm_point disjoint planet_osm_polygon -> save as "osm_addr_pts.gpkg" (takes some times)

- merge "osm_addr_ways_centroid.gpkg" + "osm_addr_pts.gpkg" -> "osm_addr.gpkg"
- remove duplicate locations

### Comparison

Add the layer municipality (from admin boundaries)

Use the tool "Count point in polygons" with the layer "municipality" and 
  - osm address pts
  - best address pts

Use the field calculator to compute :

- the ratio: NUMPOINTS_OSM/NUMPOINTS_BEST
- the diff: NUMPOINTS_OSM - NUMPOINTS_BEST

## Maps

In QGIS, 2 maps in the map layout:

- count: a map showing the ratio of OSM addresses over BEST addresses
- diff: a map showing where addresses are missing in terms of a pie diagram



