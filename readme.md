# MapboxGL Ocean Map

This is a collection of notes taken during the process of experimenting with MapboxGL, PBF Vector Tiles, and ENC data from NOAA.

## Links

* Serving Tiles
  * [Stack Exchange: Hosting Mapbox Vector Tiles](http://gis.stackexchange.com/questions/125037/self-hosting-mapbox-vector-tiles)
* MapboxGL General
  * [JustinMiller: Anatomy of a travel map](http://justinmiller.io/posts/2015/01/20/anatomy-of-a-travel-map/)
  * [Igor Tihonov: Taking mapbox-gl.js for a spin](http://igortihonov.com/2014/10/21/taking-mapbox-gl-for-a-spin/)

## Steps

### Data

At its core, it's about data. You'll need to be able to understand the data that you have and import it into your data store (ie PostGIS).

#### Links
* [Processing S57 soundings](http://blog.perrygeo.net/2005/12/03/hello-world/)
* [GDAL S57 Driver](http://www.gdal.org/drv_s57.html) (section about soundings especially important)


#### Import the data:

Below is an example of importing only the `SOUNDG` layer (soundings) of a single S57 file.
``` bash
export OGR_S57_OPTIONS="RETURN_PRIMITIVES=ON,RETURN_LINKAGES=ON,LNAM_REFS=ON,SPLIT_MULTIPOINT=ON,ADD_SOUNDG_DEPTH=ON"
ogr2ogr \
  -append \
  -nlt POINT25D \
  -f PostgreSQL PG:"dbname=s57 user=anthony" \
  US5WA70M.000 SOUNDG
```

About the options:

``` bash
-update  # Open existing output datasource in update mode rather than trying to create a new one
-append  # Append to existing layer instead of creating new
-nlt type:
   Define the geometry type for the created layer. One of NONE, GEOMETRY, POINT, LINESTRING,
   POLYGON, GEOMETRYCOLLECTION, MULTIPOINT, MULTIPOLYGON or MULTILINESTRING. Add '25D' to the name
   to get 2.5D versions.
```
Below is an example of iterating through a collection of files and importing just the `SOUNDG` layers.
``` bash
export OGR_S57_OPTIONS="RETURN_PRIMITIVES=ON,RETURN_LINKAGES=ON,LNAM_REFS=ON,SPLIT_MULTIPOINT=ON,ADD_SOUNDG_DEPTH=ON"

find . -name "US*.000" -exec ogr2ogr -append -nlt POINT25D -f PostgreSQL PG:"dbname=s57 user=anthony" {} SOUNDG \;
```
Below is an example of importing an entire S57 file.
``` bash
export OGR_S57_OPTIONS="RETURN_PRIMITIVES=ON,RETURN_LINKAGES=ON,LNAM_REFS=ON,SPLIT_MULTIPOINT=ON,ADD_SOUNDG_DEPTH=ON"

ogr2ogr -append -f PostgreSQL PG:"dbname=s57_2 user=anthony" US1HA02M.000 -skipfailures \;
```
Tying it all together, below is an example of searching recursively through a collection of S57 files and importing all layers into PostGIS.
``` bash
export OGR_S57_OPTIONS="RETURN_PRIMITIVES=ON,RETURN_LINKAGES=ON,LNAM_REFS=ON,SPLIT_MULTIPOINT=ON,ADD_SOUNDG_DEPTH=ON"

find . -name "US*.000" -exec ogr2ogr -append -f PostgreSQL PG:"dbname=enc user=anthony" {} -skipfailures \;
```

### Visualization

1. Set up a basic page after reading [this tutorial](https://gist.github.com/mapmeld/8866414b7fc8940e8540).  `index0.html`
1. How do you serve it?  [Implementations](https://github.com/mapbox/vector-tile-spec/wiki/Implementations), [node-mapnik](https://github.com/mapnik/node-mapnik), [tutorial](http://www.sparkgeo.com/labs/big/), [mapnik vectortile api](https://github.com/mapnik/node-mapnik/blob/master/docs/VectorTile.md), [vector-tile-server](https://github.com/artemp/vector-tile-server)

### Mapbox Studio

1. Create an input layer: [adding-layers](https://www.mapbox.com/tilemill/docs/manual/adding-layers/), [quick-start](https://www.mapbox.com/mapbox-studio/source-quickstart/)

    ``` bash
    (
      SELECT
        depth,
        scamin,
        scamax,
        sorind,
        sordat,
        wkb_geometry
      FROM soundg
    ) as data
    ```
1. Annotate data (see [S-57 info](http://www.caris.com/S-57/frames/S57catalog.htm), [GDAL S-57](http://gdal.org/1.11/ogr/drv_s57.html))
1. Build scale system. [source](http://msdn.microsoft.com/en-us/library/bb259689.aspx)

    ``` python
    scale = lambda x: 295829355.45 / (2**(x-1))
    for i in xrange(1, 23):
        rule = "[zoom>={zoom}][scamin>{scale}],"
        print rule.format(zoom=i, scale=scale(i))
    ```
