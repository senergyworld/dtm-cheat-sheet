# dtm-cheat-sheet
Code snippets for processing and analysing marine DTM datasets

This workflow requires a gridded 3 column XYZ file.

*2m gridded XYZ file example:*

```
408825.00 5694465.00 -33.759000
408827.00 5694465.00 -33.778000
408829.00 5694465.00 -37.711000
408831.00 5694465.00 -37.718000
408833.00 5694465.00 -37.730000
408835.00 5694465.00 -37.750000
408837.00 5694465.00 -37.705000
```

Required tools for installation:

- [OSGeo4W](https://trac.osgeo.org/osgeo4w/)
- [Generic Mapping Tools (GMT)](http://gmt.soest.hawaii.edu/projects/gmt/wiki/Download)

## WINDOWS

Check the resolution of the XYZ file in a text editor, if the XYZ file is too big to be opened by a text editor then open via the Windows command prompt.

`more input.xyz`

Run `gmtinfo` and pipe  the output dimensions of the input XYZ grid to a text file

`gmtinfo input.xyz -I1 > dimensions.txt`

- where `-I` is cell size of the XYZ file eg. 1m x 1m

#### Convert XYZ to GeoTIFF

Use `xyz2grd` to convert a standard 3-column gridded XYZ to GeoTIFF

`xyz2grd -R0/1162/0/1246 -I2 -Goutput.tif=gd:GTiff input.xyz`

- where `-R` are the output grid dimensions from `gmtinfo`

- where `-I` is cell size of the XYZ file

- where `-G` is output file name of the GeoTiff

Note |  for sub metre resolution, eg. `-I0.1` for 10cm cell size

#### Invert Z column from positive to negative values

`grdmath input.tif=gd:GTiff -1 MUL = output.tif=gd:GTiff`

#### Apply CRS & Compression

Use `gdal_translate` to apply the coordinate reference system and high compression

`gdal_translate -a_srs EPSG:23031 -co COMPRESS=DEFLATE -co PREDICTOR=2 -co ZLEVEL=9 input.tif output.tif`

- use `-a_srs` to overide projection, recommend using [www.epsg.io](www.epsg.io)

- use `-co` to apply compression when creating the output GeoTiff

#### Generate Hillshade

`gdaldem hillshade input.tif output_hillshade.tif -z [Zfactor] -az [Azimuth (default 315)] -alt [altitude (default 45)]`

- where `-z` is the vertical exaggeration or Zfactor
- where `-az` is the azimuth angle in degrees
- where `-alt` is the altitude of the light, in degrees

#### Generate Seabed Gradient

`gdaldem slope input.tif output_slope.tif`

#### Generate color-relief

`gdaldem color-relief bathy_merged.tif hillshade_colours.txt color-relief.tif -alpha -nearest_color_entry`

##### Format for color-relief input file
- Delimited file (comma, tab or space)
- Four columns | elevation, Red, Green, Blue

You can use common colours as used in [GRASS](https://grass.osgeo.org/grass64/manuals/r.colors.html) instead of RGB values.

Example #1 hardcoded water depth break values, with associated RGB colour

```
0 0 0 0
-14.2290 182 0 208
-16.0592 134 0 209
-17.8894 80 0 211
-19.7196 25 0 213
-21.5498 1 30 215
```

Example #2 Can also use percentages: 0% minimum > 100% maximum value in raster
```
0% 0 0 0
20% 182 0 208
40% 134 0 209
60% 80 0 211
80% 25 0 213
100% 1 30 215
```
http://www.gdal.org/gdaldem.html for more info

#### Generate Contours

`gdal_contour -a elev -i 1 input.tif output.shp`

- where `-a` provides the attribute name for the elevation

- where `-i` is the elevation interval between contours

#### Creating Hillshade Relief Images

*Combining, color-relief and hillshade into one image*

Method #1 | You can use Frank Warmerdam [hsv_merge.py](http://svn.osgeo.org/gdal/trunk/gdal/swig/python/samples/hsv_merge.py) script

`hsv_merge.py color-relief.tif hillshade.tif output.tif`

#### Create a polygon coverage shapefile of a raster area
To select all values in input raster. Set the logic test to be `A>-100` (all values are > -100) and set them to equal an integer i.e. 1

`gdal_calc.py -A input.tif --outfile=myinteger.tif --calc="1*(A>-100)" --overwrite`

Polygonise the resulting integer tiff to a shape file

`gdal_polygonize.bat myinteger.tif -f "ESRI Shapefile" output.shp`
