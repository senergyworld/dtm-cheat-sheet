# dtm-cheat-sheet

## Introduction

This is a cheat-sheet for various code snippets to assist with processing and analysing marine digital terrain model (DTM) bathymetric datasets. In the marine sector(s), a common filetype for a bathymetric DTM is the 3-column gridded XYZ file, following the structure below. Note, in marine gridded XYZ, it is common to _not_ include `no data` values within the XYZ file. Because of this, there can be issues with using the XYZ driver in GDAL/OGR. However, `gmt` seems to read marine XYZ files consistently.  
*2m standard 3-column gridded XYZ file example:*
```
408825.00 5694465.00 -33.759000
408827.00 5694465.00 -33.778000
408829.00 5694465.00 -37.711000
408831.00 5694465.00 -37.718000
408833.00 5694465.00 -37.730000
408835.00 5694465.00 -37.750000
408837.00 5694465.00 -37.705000
```

### Prerequisites:
* The following workflows use a combination of specialist programs, sourced from two software libraries called [GDAL/OGR](http://www.gdal.org/) and  [gmt](http://gmt.soest.hawaii.edu/projects/gmt/wiki/Download).  
* Each library contains a collection of specific programs designed to do one thing only.
* For instance, within the `GDAL/OGR` library, there is a specific program called `gdalinfo`. This program interogrates a raster or gridded file and generates a short report on the structure and metadata about that raster file.
* All programs are executed on the `commandline`, which is also sometimes referred to as the `shell`, `console` or `terminal`.
* If you're unfamiliar with using commandline programs, the command or program is ususal referenced first, then followed by various input parameters. Input parameters can be mandatory or optional, depending on the particular program.

### Installation of `gmt`
* provide link to installation instructions
 
### Installation of `GDAL/OGR`
* provide link to install instructions

## WINDOWS

Run the following commands via the `OSGeo4W Shell` commonly found here `C:\OSGeo4W\OSGeo4W.bat`

#### Analyse XYZ

Check the resolution of the XYZ file in a text editor, if the XYZ file is too big to be opened by a text editor then open via the Windows command prompt.

```
more input.xyz
```

Run `gmtinfo` and pipe  the output dimensions of the input XYZ grid to a text file

```
gmtinfo input.xyz -I2 > dimensions.txt
```
- where `-I` is the cell size of the XYZ file, eg `-I2` for a 2x2m cell size

Ascertain Z range of the input XYZ file:

```
gmtinfo -C -o4,5 input.xyz
```

- where `-C` reports the min/max values per column, eg (X,Y and Z)

- where `-o` limits the output columns reported. `4,5` report the min/max for the 3rd column (Z column)

#### Convert XYZ to GeoTIFF

Use `gmt` `xyz2grd` to convert a standard 3-column gridded XYZ to GeoTIFF

```
xyz2grd -R399073.9/409593.9/5694455.6/5702819.6 -I2 -Goutput.tif=gd:GTiff input.xyz
```

- where `-R` are the output grid dimensions from `gmtinfo`

- where `-I` is the cell size of the XYZ file, eg `-I2` for a 2x2m cell size

- where `-G` is output file name of the GeoTiff. Use the suffix `=gd:GTiff` to force the output to a GeoTiff. Default output file format is netCDF.

Note | Use decimals for sub metre resolution, eg. `-I0.1` for 10cm cell size

#### Invert Z column from positive to negative values

If your input XYZ reports the Z value in positive depths as opposed to negative elevation, you can invert using `gmt` `grdmath`

`grdmath input.tif=gd:GTiff -1 MUL = output_inverted.tif=gd:GTiff`

#### Apply CRS & Compression

Use `gdal_translate` to apply the coordinate reference system and high compression

`gdal_translate -a_srs EPSG:23031 -co COMPRESS=DEFLATE -co PREDICTOR=2 -co ZLEVEL=9 input_frm_xyz2grd.tif output_bathymetry.tif`

- use `-a_srs` to overide projection, recommend using [www.epsg.io](www.epsg.io)

- use `-co` to apply compression when creating the output GeoTiff. You can use multiple `-co` options.

#### Generate Hillshade

`gdaldem hillshade input_bathymetry.tif output_hillshade.tif -z [Zfactor (default 1)] -az [Azimuth (default 315)] -alt [altitude (default 45)] -co COMPRESS=DEFLATE -co PREDICTOR=2 -co ZLEVEL=9`

- where `-z` is the vertical exaggeration or Zfactor

- where `-az` is the azimuth angle in degrees

- where `-alt` is the altitude of the light, in degrees

- use `-co` to apply compression when creating the output GeoTiff. You can use multiple `-co` options.

#### Generate Seabed Gradient

`gdaldem slope input_bathymetry.tif output_slope.tif -co COMPRESS=DEFLATE -co PREDICTOR=2 -co ZLEVEL=9`

- use `-co` to apply compression when creating the output GeoTiff. You can use multiple `-co` options.

#### Generate Color-Relief

`gdaldem color-relief input_bathymetry.tif color-relief-ramp.txt output_color-relief.tif -alpha -nearest_color_entry -co COMPRESS=DEFLATE -co PREDICTOR=2 -co ZLEVEL=9`

- `-alpha` will add an alpha channel to the output raster

- `-nearest_color_entry` will use the closest RGBA value. Use `-exact_color_entry` for strict color matching.

- use `-co` to apply compression when creating the output GeoTiff. You can use multiple `-co` options.

###### Format for color-relief-ramp input file

- Delimited file (comma, tab or space)

- Four columns | elevation, Red, Green, Blue

You can use common colours as used in [GRASS](https://grass.osgeo.org/grass64/manuals/r.colors.html) instead of RGB values.

Example #1 hardcoded water depth break values, with associated RGB colour

```
nv 0 0 0
-44.302 182 0 208
-42.681 134 0 209
-41.06  80  0 211
-39.439 25  0 213
-37.818 1 30  215
-36.197 0 87  217
-34.576 1 140 218
-32.955 0 199 220
-31.334 0 222 184
-29.713 0 224 129
-28.092 2 226 71
-26.471 2 228 17
-24.85  46  229 2
-23.229 106 231 1
-21.608 167 233 1
-19.987 229 235 2
-18.366 237 185 3
-16.745 239 126 3
-15.124 240 64  2
-13.503 242 4 2
-11.882 244 2 61
```

Example #2 Can also use percentages: 0% minimum > 100% maximum value in raster
```
nv 255 255 255
0% 0 0 0
10% 29 43 53
20% 37 44 95
30% 63 70 134
40% 89 112 147
50% 87 124 143
60% 117 160 125
70% 188 219 173
80% 239 253 163
90% 222 214 67
100% 189 138 55
```
http://www.gdal.org/gdaldem.html for more info

#### Generate Contours

`gdal_contour -a elev -i 1 input_bathymetry.tif output_contour_1m.shp`

- where `-a` provides the attribute name for the elevation

- where `-i` is the elevation interval between contours

#### Creating Hillshade Relief Images

*Combining, color-relief and hillshade into one image*

Method #1 | You can use Frank Warmerdam [hsv_merge.py](http://svn.osgeo.org/gdal/trunk/gdal/swig/python/samples/hsv_merge.py) script

`python hsv_merge.py output_color-relief.tif output_hillshade.tif final_output_shaded_relief.tif`

#### Create a polygon coverage shapefile of a raster area
To select all values in input raster. Set the logic test to be `A>-100` (all values are > -100) and set them to equal an integer i.e. 1

`gdal_calc.py -A input_bathymetry.tif --outfile=myinteger.tif --calc="1*(A>-100)" --overwrite`

Polygonise the resulting integer tiff to a shape file

`gdal_polygonize.bat myinteger.tif -f "ESRI Shapefile" output_bathymetry_coverage.shp`

#### Convert GeoTiff to standard 3-column gridded XYZ file

`grd2xyz -s input_bathymetry.tif > output_grd2xyz_bathymetry.xyz`

* use `-s` to suppress output for records whose z-value equals NaN [Default outputs all records]
