# pyshepseg.subset
    This module contains functionality for subsetting a large
    segmented image into a smaller one for checking etc. See
    :func:`subsetImage`.

## Classes
### class PyShepSegSubsetError
      Common base class for all non-exit exceptions.

## Functions
### def copyColumns(inRat, outRat)
        Copy column structure from inRat to outRat. Note that this just creates
        the empty columns in outRat, it does not copy any data.
        
        Parameters
        ----------
          inRat, outRat : gdal.RasterAttributeTable
            Columns found in inRat are created on outRat
        
        Returns
        -------
          numIntCols : int
            Number of integer columns found
          numFloatCols : int
            Number of float columns found

### def readRATIntoPage(rat, numIntCols, numFloatCols, minVal, maxVal)
        Create a new RatPage() object that represents the section of the RAT
        for a tile of an image. The part of the RAT between minVal and maxVal
        is read in and a RatPage() instance returned with the startSegId param
        set to minVal.

### def subsetImage(inname, outname, tlx, tly, newXsize, newYsize, outformat, creationOptions=[], origSegIdColName=None, maskImage=None)
        Subset an image and "compress" the RAT so only values that
        are in the new image are in the RAT. Note that the image values
        will be recoded in the process.
        
        gdal_translate seems to have a problem with files that
        have large RAT's so while that gets fixed do the subsetting
        in this function.
        
        Parameters
        ----------
          inname : str or gdal.Dataset
            Filename of input raster, or open gdal Dataset object
          outname : str
            Filename of output raster
          tlx, tly : int
            The x & y pixel coordinates (i.e. col & row, respectively) of the
            top left pixel of the image subset to extract.
          newXsize, newYsize : int
            The x & y size (in pixels) of the image subset to extract
          outformat : str
            The GDAL short name of the format driver to use for the output
            raster file
          creationOptions : list of str
            The GDAL creation options for the output file
          origSegIdColName : str or None
            The name of a RAT column. If not None, this will be created
            in the output file, and will contain the original segment ID
            numbers so the new segment IDs can be linked back to the old
          maskImage : str or None
            If not None, then the filename of a mask raster. Only pixels
            which are non-zero in this mask image will be included in the
            subset. This image is assumed to match the shape and position
            of the output subset

### def writeCompletedPagesForSubset(inRAT, outRAT, outPagedRat)
        For the subset operation. Writes out any completed pages to outRAT
        using the inRAT as a template.
        
        Parameters
        ----------
          inRAT, outRAT : gdal.RasterAttributeTable
            The input and output raster attribute tables.
          outPagedRat : numba.typed.Dict
            The paged RAT in memory, as created by createPagedRat()

## Values
    copySubsettedSegmentsToNew = CPUDispatcher(<function copySubsettedSegmentsToNew at 0x743505ed9260>)
    processSubsetTile = CPUDispatcher(<function processSubsetTile at 0x743505eda020>)
    readColDataIntoPage = CPUDispatcher(<function readColDataIntoPage at 0x743505ed99e0>)
    setHistogramFromDictionary = CPUDispatcher(<function setHistogramFromDictionary at 0x743505ed9300>)
    setSubsetRecodeFromDictionary = CPUDispatcher(<function setSubsetRecodeFromDictionary at 0x743505ed96c0>)
