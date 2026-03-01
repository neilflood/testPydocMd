# rios.fileinfo
    Utility classes for accessing information from files, ouside of the
    main RIOS applier structure. Typically these are used to access information
    required to set up the call to :func:`rios.applier.apply`, passing some of the 
    information in via the otherargs parameter. 

## Classes
### class ColumnStats
      Summary statistics for a single column of a raster attribute table
      
      Attributes are:
          * **count**
          * **mean**
          * **stddev**
          * **min**
          * **max**
          * **sum**
          * **mode**            Not yet implemented
          * **median**          Not yet implemented
      
      Unless otherwise requested, the statistics are weighted by the Histogram column, so
      that they represent spatial statistics, e.g. the mean would correspond to the mean
      over all pixels, rather than the mean over all rows, and the count is effectively
      the number of pixels, rather than the number of rows. This can be disabled by
      setting histogramweighted=False when calling the constructor. 
      
      Unless otherwise requested, rows which correspond to the image null value are not
      included in stats calculation. This can be reversed by setting includeImageNull=True
      on the constructor, although normally the histogram does not include counts for the 
      null value either, so this only really makes sense when using histogramweighted=False. 
      (N.B. that last point assumes the presence of the GDAL patches for 
      tickets #4750 and #5289. )
          

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ColumnStats.\_\_init\_\_(self, band, columnName, includeImageNull=False, histogramweighted=True)
          Initialize self.  See help(type(self)) for accurate signature.

### class ImageFileStats
      Hold the stats for all layers in an image file. This object can be indexed 
      with the layer index, and each element is an instance of :class:`rios.fileinfo.ImageLayerStats`. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageFileStats.\_\_init\_\_(self, filename)
          Initialize self.  See help(type(self)) for accurate signature.

### class ImageInfo
      An object with the bounds and other info for the given image, 
      in GDAL conventions. 
      
      Object contains the following fields
          * **xMin**            Map X coord of left edge of left-most pixel
          * **xMax**            Map X coord of right edge of right-most pixel
          * **yMin**            Map Y coord of bottom edge of bottom pixel
          * **yMax**            Map Y coord of top edge of top-most pixel
          * **xRes**            Map coord size of each pixel, in X direction
          * **yRes**            Map coord size of each pixel, in Y direction
          * **nrows**           Number of rows in image
          * **ncols**           Number of columns in image
          * **transform**       Transformation params to map between pixel and map coords, in GDAL form
          * **projection**      WKT string of projection
          * **rasterCount**     Number of rasters in file
          * **lnames**          Names of the layers as a list.
          * **layerType**       "thematic" or "athematic", if it is set
          * **dataType**        Data type for the first band (as a GDAL integer constant)
          * **dataTypeName**    Data type for the first band (as a human-readable string)
          * **nodataval**       Value used as the no-data indicator (per band)
      
      The omitPerBand argument on the constructor is provided in order to speed up the 
      access of very large VRT stacks. The information which is normally extracted 
      from each band will, in that case, trigger a gdal.Open() for each band, which 
      can be quite slow. So, if none of that information is actually required, then 
      setting omitPerBand=True will omit that information, but will return as quickly 
      as for a normal single file. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageInfo.\_\_init\_\_(self, filename, omitPerBand=False)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageInfo.getCorners(self, outWKT=None, outEPSG=None, outPROJ4=None)
          Return the coordinates of the image corners, possibly reprojected. 
          
          This is the same information as in the xMin, xMax, yMin, yMax fields, 
          but with the option to reproject them into a given output projection. 
          Because the output coordinate system will not in general align with the 
          image coordinate system, there are separate values for all four corners. 
          These are returned as::
          
              (ul_x, ul_y, ur_x, ur_y, lr_x, lr_y, ll_x, ll_y)
              
          The output projection can be given as either a WKT string, an 
          EPSG number, or a PROJ4 string. If none of those is given, then 
          bounds are not reprojected, but will be in the same coordinate 
          system as the image corners. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageInfo.layerNameFromNumber(self, layerNumber)
          Return the layer name corresponding to the given layer number. 
          Valid layer numbers are as per GDAL conventions, i.e. starting at 1.
          If the given layer number is not valid for this file info, an exception is 
          raised. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageInfo.layerNumberFromName(self, layerName)
          Return the layer number corresponding to the given layer name.
          Valid layer numbers are as per GDAL conventions, i.e. starting at 1. 
          If the given layer name is not found in this file info, then zero is returned. 

### class ImageLayerStats
      Hold the stats for a single image layer. These are as retrieved
      from the given image file, and are not calculated again. If they
      are not present in the file, they will be None. 
      Typically this class is not used separately, but only instantiated 
      as a part of the ImageFileStats class. 
      
      The object contains the following fields
          * **mean**        Mean value over all non-null pixels
          * **min**         Minimum value over all non-null pixels
          * **max**         Maximum value over all non-null pixels
          * **stddev**      Standard deviation over all non-null pixels
          * **median**      Median value over all non-null pixels
          * **mode**        Mode over all non-null pixels
          * **skipfactorx** The statistics skip factor in the X direction
          * **skipfactory** The statistics skip factor in the Y direction
          
      There are many ways to report a histogram. 
      The following attributes report it the way GDAL does. 
      See GDAL doco for precise details. 
      
          * **histoCounts**     Histogram counts (numpy array)
          * **histoMin**        Minimum edge of smallest bin
          * **histoMax**        Maximum edge of largest bin
          * **histoNumBins**    Number of histogram bins
          

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageLayerStats.\_\_init\_\_(self, bandObj)
          Initialize self.  See help(type(self)) for accurate signature.

### class RatStats
      Calculate statistics on columns in a Raster Attribute Table
      
      Normal usage is via the RatStats class, e.g.::
      
          columnsOfInterest = ['col1', 'col4']
          ratStatsObj = RatStats('file.img', columnlist=columnsOfInterest)
      
          print ratStatsObj.col1.mean, ratStatsObj.col4.mean
      
      Contains an attribute for each named column. Each attribute is 
      a ColumnStats object, containing all relevant global stats 
      for that column. See docstring for that class for details. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatStats.\_\_init\_\_(self, filename, columnlist=None, layernum=1, includeImageNull=False, histogramweighted=True)
          Default columnlist is all columns in the table. 

### class VectorFileInfo
      Hold useful general information about a vector file. This object
      can be indexed with the layer index, and each element is
      an instance of :class:`rios.fileinfo.VectorLayerInfo`. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; VectorFileInfo.\_\_init\_\_(self, filename)
          Initialize self.  See help(type(self)) for accurate signature.

### class VectorLayerInfo
      Hold useful general information about a single vector layer. 
      
      Object contains the following fields
          * **featureCount**        Number of features in the layer
          * **xMin**                Minimum X coordinate
          * **xMax**                Maximum X coordinate
          * **yMin**                Minimum Y coordinate
          * **yMax**                Maximum Y coordinate
          * **geomType**            OGR geometry type code (integer)
          * **geomTypeStr**         Human-readable geometry type name (string)
          * **fieldCount**          Number of fields (i.e. columns) in attribute table
          * **fieldNames**          List of names of attribute table fields
          * **fieldTypes**          List of the type code numbers of each attribute table field
          * **fieldTypeNames**      List of the string names of the field types
          * **spatialRef**          osr.SpatialReference object of layer projection

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; VectorLayerInfo.\_\_init\_\_(self, ds, i)
          Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def preventGdal3axisSwap(sr)
        Guard the given spatial reference object against axis swapping, when
        running with GDAL 3. Does nothing if GDAL < 3. Modifies the
        object in place.

## Values
    geometryTypeStringDict = {1: 'Point', 2: 'Line', 3: 'Polygon'}
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
