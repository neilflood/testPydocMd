# pyshepseg.subset
    This module contains functionality for subsetting a large
    segmented image into a smaller one for checking etc. See
    :func:`subsetImage`.

## Classes
### class PyShepSegSubsetError(Exception)
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

### def copySubsettedSegmentsToNew(inPage, outPagedRat, recodeDict)
        Using the recodeDict, copy across the rows inPage to outPage.

        inPage is processed and (taking into account of inPage.startSegId)
        the original input row found. This value is then
        looked up in recodeDict to find the row in the output RAT to
        copy the row from the input to.

        Parameters
        ----------
          inPage : tilingstats.RatPage
            A page of RAT from the input file
          outPagedRat : numba.typed.Dict
            In-memory pages of the output RAT, as created by createPagedRat().
            This is modified in-place, creating new pages as required.
          recodeDict : numba.typed.Dict
            Keyed by original segment ID, values are the corresponding
            segment IDs in the subset

### def processSubsetTile(tile, recodeDict, histogramDict, maskData)
        Process a tile of the subset area. Returns a new tile with the new codes.
        Fills in the recodeDict as it goes and also updates histogramDict.

        Parameters
        ----------
          tile : shepseg.SegIdType ndarray (tileNrows, tileNcols)
            Input tile of segment IDs
          recodeDict : numba.typed.Dict
            Keyed by original segment ID, values are the corresponding
            segment IDs in the subset
          histogramDict : numba.typed.Dict
            Histogram counts in the subset, keyed by new segment ID
          maskData : None or int ndarray (tileNrows, tileNcols)
            If not None, then is a raster mask. Only pixels which are
            non-zero in the mask will be included in the subset

        Returns
        -------
          outData : shepseg.SegIdType ndarray (tileNrows, tileNcols)
            Recoded copy of the input tile.

### def readColDataIntoPage(page, data, idx, colType, minVal)
        Numba function to quickly read a column returned by
        rat.ReadAsArray() info a RatPage.

### def readRATIntoPage(rat, numIntCols, numFloatCols, minVal, maxVal)
        Create a new RatPage() object that represents the section of the RAT
        for a tile of an image. The part of the RAT between minVal and maxVal
        is read in and a RatPage() instance returned with the startSegId param
        set to minVal.

### def setHistogramFromDictionary(dictn, histArray)
        Given a dictionary of pixel counts keyed on index,
        write these values to the array.

### def setSubsetRecodeFromDictionary(dictn, array)
        Given the recodeDict write the original values to the array
        at the new indices.

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

