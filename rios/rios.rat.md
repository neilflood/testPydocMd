# rios.rat
    This module contains routines for reading and writing Raster
    Attribute Tables (RATs). These are designed to be able to 
    be called from outside of RIOS.
    
    Within RIOS, these are called from the :class:`rios.imagewriter.ImageWriter 
    and :class:`rios.fileinfo.ImageInfo` classes.
    
    **Note:** It is recommended that the newer :mod:`rios.ratapplier` module be used instead of this
    interface for large tables where possible.

## Classes
### class str
      str(object='') -> str
      str(bytes_or_buffer[, encoding[, errors]]) -> str
      
      Create a new string object from the given object. If encoding or
      errors is specified, then the object must expose a data buffer
      that will be decoded using the given encoding and error handler.
      Otherwise, returns the result of object.__str__() (if defined)
      or repr(object).
      encoding defaults to sys.getdefaultencoding().
      errors defaults to 'strict'.

## Functions
### def genColorTable(numEntries, colortype)
        Generate a colour table array. The type of colour table generated
        is controlled by the colortype string. Possible values are:
        
            * "rainbow"
            * "gray"
            * "random"
        
        See corresponding genColorTable_<colortype> function for details of each. 

### def genColorTable_gray(numEntries)
        Generate a color table array with the given number of entries, with all 
        colors being shades of grey. First entry is black, last entry is white. 
        
        Returns the array as a 4 column array, suitable for use with the 
        :func:`rios.rat.setColorTable` function. 

### def genColorTable_rainbow(numEntries)
        Generate a color table array with the given number of entries, with colors notionally
        describing a rainbow (i.e. red-orange-yellow-green-blue-indigo-violet). Probably
        not what a painter would call a rainbow, but it will do. 
        
        Returns the array as a 4 column array, suitable for use with the 
        :func:`rios.rat.setColorTable` function. 

### def genColorTable_random(numEntries)
        Generate a color table array with the given number of entries by assigning 
        random red/green/blue values. No attempt is made to always generate
        unique colours, i.e. it is randomly possibly for different entries to
        have the same colour. 
        
        Returns the array as a 4 column array, suitable for use with the 
        :func:`rios.rat.setColorTable` function. 

### def getColorTable(imgFile, bandNumber=1)
        Given either an open gdal dataset, or a filename,
        reads the color table as an array that can be passed
        to rat.setColorTable()
        
        The returned colour table is a numpy array, described in detail
        in the docstring for rat.setColorTable(). 

### def getColumnNames(imgFile, bandNumber=1)
        Given either an open gdal dataset, or a filename,
        Return the names of the columns in the attribute table
        associated with the file as a list.

### def getColumnNamesFromBand(gdalBand)
        Return the names of the columns in the attribute table
        associated with the gdalBand as a list.

### def getUsageOfColumn(imgFile, colName, bandNumber=1)
        Given either an open gdal dataset, or a filename,
        returns the 'usage' of the column which can then be passed
        to writeColumn to preserve usage when copying

### def getUsageOfColumnFromBand(gdalBand, colName)
        Given a gdalBand returns the usage of the named column

### def inferColumnType(sequence)
        Infer from the type of the first element in the sequence

### def isColorColFromUsage(usage)
        Tells if usage is one of the color column types

### def readColumn(imgFile, colName, bandNumber=1, useStringDType=False)
        Given either an open gdal dataset, or a filename,
        extract the Raster Attribute with the
        given name. Returns a numpy array of the appropriate dtype. For
        GFT_Integer or GFT_Float columns, this is as returned by GDAL's
        ReadAsArray routine. For GFT_String columns, the default is to
        return the array of bytes as given by ReadAsArray, but if the
        useStringDType parameter is True, the array is converted to
        numpy's StringDType, i.e. an array of variable-length Unicode strings.

### def readColumnFromBand(gdalBand, colName, useStringDType=False)
        Given a GDAL Band, extract the Raster Attribute with the
        given name. Returns a numpy array of the appropriate dtype. For
        GFT_Integer or GFT_Float columns, this is as returned by GDAL's
        ReadAsArray routine. For GFT_String columns, the default is to
        return the array of bytes as given by ReadAsArray, but if the
        useStringDType parameter is True, the array is converted to
        numpy's StringDType, i.e. an array of variable-length Unicode strings.

### def setColorTable(imgfile, colorTblArray, layernum=1)
        Set the color table for the specified band. You can specify either 
        the imgfile as either a filename string or a gdal.Dataset object. The
        layer number defaults to 1, i.e. the first layer in the file. 
        
        The color table is given as a numpy array of 5 columns. There is one row 
        (i.e. first array index) for every value to be set, and the columns
        are:
        
            * pixelValue
            * Red
            * Green
            * Blue
            * Opacity
            
        The Red/Green/Blue values are on the range 0-255, with 255 meaning full 
        color, and the opacity is in the range 0-255, with 255 meaning fully 
        opaque. 
        
        The pixels values in the first column must be in ascending order, but do 
        not need to be a complete set (i.e. you don't need to supply a color for 
        every possible pixel value - any not given will default to transparent black).
        It does not even need to be contiguous. 
        
        For reasons of backwards compatability, a 4-column array will also be accepted, 
        and will be treated as though the row index corresponds to the pixelValue (i.e. 
        starting at zero). 

### def writeColumn(imgFile, colName, sequence, colType=None, bandNumber=1, colUsage=0)
        Given either an open gdal dataset, or a filename,
        writes the data specified in sequence (can be list, tuple or array etc)
        to the named column in the attribute table assocated with the
        file. colType must be one of gdal.GFT_Integer,gdal.GFT_Real,gdal.GFT_String.
        can specify one of the gdal.GFU_* constants for colUsage - default is 'generic'

### def writeColumnToBand(gdalBand, colName, sequence, colType=None, colUsage=0)
        Given a GDAL band, Writes the data specified in sequence 
        (can be list, tuple or array etc)
        to the named column in the attribute table assocated with the
        gdalBand. colType must be one of gdal.GFT_Integer,gdal.GFT_Real,gdal.GFT_String.
        can specify one of the gdal.GFU_* constants for colUsage - default is 'generic'
        GDAL dataset must have been created, or opened with GA_Update

## Values
    DEFAULT_AUTOCOLORTABLETYPE = None
    MAX_RECOMMENDED_COLOR_TABLE = 256
