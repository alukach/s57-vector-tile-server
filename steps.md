# Steps

## Data

1. Working with S-57: [Processing S57 soundings](http://blog.perrygeo.net/2005/12/03/hello-world/), [GDAL Driver (section about soundings especially important)](http://www.gdal.org/drv_s57.html)
1. Import the data:

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


## Visualization

1. Set up a basic page after reading [this tutorial](https://gist.github.com/mapmeld/8866414b7fc8940e8540).  `index0.html`
1. How do you serve it?  [Implementations](https://github.com/mapbox/vector-tile-spec/wiki/Implementations), [node-mapnik](https://github.com/mapnik/node-mapnik), [tutorial](http://www.sparkgeo.com/labs/big/), [mapnik vectortile api](https://github.com/mapnik/node-mapnik/blob/master/docs/VectorTile.md), [vector-tile-server](https://github.com/artemp/vector-tile-server)

## Mapbox Studio

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
    