# pyshepseg.tilingstats
    Routines to support calculation of statistics on large rasters.
    The statistics are calculated per segment so require two input
    rasters - the segmentation output, and another image to gather 
    statistics from, for each segment in the first image. 
    
    These are optimised to work on one tile of the images at a
    time so should be efficient in terms of memory use.
    
    The main functions in this module are :func:`calcPerSegmentStatsTiled`
    and :func:`calcPerSegmentSpatialStatsTiled`.

## Classes
### class OpenRatContainer
      Hold all data structures for an open RAT, hiding the distinction
      between GDAL-based and Zarr-based RATs. The constructor takes
      either a single RatZarr object rz, or a pair of GDAL objects
      ds & band.
      
      Parameters
      ----------
        ds : gdal.Dataset or None
          Open Dataset object
        band : gdal.Band or None
          Open band on ds
        rz : ratzarr.RatZarr or None
          Open RatZarr object

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.CreateColumn(self, colName, colType)
        Create the column with the given name and type. For GDAL RAT, always
        use GFU_Generic usage

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.SetRowCount(self, rowCount)
        Set the row count for the table

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.WriteArray(self, colArr, colNumber, start)
        Intended to look like GDAL's RAT WriteArray function.
        
        When the output RAT is a GDAL file, parameters are passed straight
        through. When using a RatZarr file, the colNdx is translated to
        a column name and the data written to that column.

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.\__init__(self, ds=None, band=None, rz=None)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.checkColType(self, colName, colType)
        Check that the given column (pre-existing) is compatible with
        the given GDAL column type (gdal.GFT_*). Raise exception if not.

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.close(self)
        Close the open file handles

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.colExists(self, colName)
        Check if the named column already exists in the RAT

#### &nbsp;&nbsp;&nbsp;&nbsp; OpenRatContainer.getColNdx(self, colName)
        Get the column index for the given name. The index is only meaningful
        for GDAL-based RAT, but we fake it for Zarr-based, so we can
        continue to use it as the basic identifier in the numba-compiled
        sections of code (i.e. the statsSelection_fast structure).

### class PyShepSegStatsError
      Common base class for all non-exit exceptions.

### class RatPage
      Hold a single page of the paged RAT

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.\__init__(self, numIntCols, numFloatCols, startSegId, numSeg)
        Allocate arrays for int and float columns. Int columns are
        stored as signed int32, floats are float32. 
        
        startSegId is the segment ID number of the lowest segment in this page.
        numSeg is the number of segments within this page, normally the
        page size, but the last page will be smaller. 
        
        numIntCols and numFloatCols are as returned by makeFastStatsSelection().

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.getIndexInPage(self, segId)
        Return the index for the given segment, within the current
        page. 

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.getRatVal(self, segId, colType, colArrayNdx)
        Get the RAT entry for the given segment.

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.getSegmentComplete(self, segId)
        Returns True if the segment has been flagged as complete

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.pageComplete(self)
        Return True if the current page has been completed

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.setRatVal(self, segId, colType, colArrayNdx, val)
        Set the RAT entry for the given segment,
        to be the given value. 

#### &nbsp;&nbsp;&nbsp;&nbsp; RatPage.setSegmentComplete(self, segId)
        Flag that the given segment has had all stats calculated. 

### class SegPoint
      Class for handling a given data point and it's location
      in pixel space (within the whole image, not a tile).
      
      Used so that all the data for a given segment can be collected
      together even if the segment straddles a tile.

#### &nbsp;&nbsp;&nbsp;&nbsp; SegPoint.\__init__(self, x, y, val)
        Initialize self.  See help(type(self)) for accurate signature.

### class SegmentStats
      Manage statistics for a single segment

#### &nbsp;&nbsp;&nbsp;&nbsp; SegmentStats.\__init__(self, segmentHistDict, missingStatsValue)
        Construct with generic statistics, given a typed 
        dictionary of the histogram counts of all values
        in the segment.
        
        If there are no valid pixels then the value passed
        in as missingStatsValue is returned for the requested
        stats.

#### &nbsp;&nbsp;&nbsp;&nbsp; SegmentStats.getPercentile(self, percentile)
        Return the pixel value for the given percentile, 
        e.g. getPercentile(50) would return the median value of 
        the segment

#### &nbsp;&nbsp;&nbsp;&nbsp; SegmentStats.getStat(self, statID, param)
        Return the requested statistic

### class StatsReadConfig
      Set up configuration information for running read workers
      
      Parameters
      ----------
        numWorkers : int
          Number of read workers to use. If zero, reading of each tile is
          done sequentially with processing.
        bufferInsertTimeout : int
          Number of seconds to wait to insert tile data into read buffer.
          Only relevant if using read workers.
        bufferPopTimeout : int
          Number of seconds to wait to get tile data from read buffer.
          Only relevant if using read workers.

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadConfig.\__init__(self, numWorkers=0, bufferInsertTimeout=60, bufferPopTimeout=60)
        Initialize self.  See help(type(self)) for accurate signature.

### class StatsReadManager
      Open the input imgfile and segfile, optionally starting some
      read workers. It is assumed that the two rasters have same size/shape
      and pixel alignment.
      
      Parameters
      ----------
        imgfile : str or gdal.Dataset
          Name or open gdal.Dataset of the imagery on which to collect
          statistics.
        imgbandnum : int
          Band number (starts at 1) of imgfile on which to collect statistics
        segfile : str or gdal.Dataset
          Name or open gdal.Dataset of segmentation raster. This file
          has the segment ID of each pixel.
        segbandnum : int
          Band number (starts at 1) of the band in segfile for the RAT.
          Default is 1 (the usual case).
        readCfg : Instance of StatsReadConfig
          Configuration of read workers.
        tileSize : int
          Size (in pixels) of tiles i.e. shape is (tileSize, tileSize)
        numXtiles : int
          Number of tiles in X direction across the images
        numYtiles : int
          Number of tiles in Y direction across the images
        timings : Timers
          A Timers object in which read timings are recorded. Default will
          discard timings.

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadManager.\__init__(self, imgfile, imgbandnum, segfile=None, segbandnum=1, segband=None, readCfg=None, tileSize=None, numXtiles=None, numYtiles=None, timings=None)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadManager.close(self)
        Close the GDAL objects

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadManager.popNextTile(self)
        Get the data for the next tile. If using read workers, pop the next
        available tile out of the buffer, otherwise just read in directly
        from the files.
        
        In the buffer case, note that we may lose the strict tile ordering,
        if a tile is available out of normal sequence. This is not generally
        a serious problem, and avoids a potential deadlock condition if we
        attempted to adhere to a strict order but read workers delivered them
        a long way out of order. Mostly would not be a problem, but serious if
        if it did occur.

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadManager.readTile(self, segBand, imgBand, tileRow, tileCol)
        Read a single tile from the two input rasters. The tile row/col
        numbers refer to a grid of tiles, so the first row of tiles
        is row 0, the second row is row 1, etc.
        
        Parameters
        ----------
          segBand, imgBand : gdal.Band
            GDAL Band objects for segmentation and image rasters
          tileRow : int
            Row number of requested tile
          tileCol : int
            Col number of requested tile
        
        Returns
        -------
          tilePair : tuple of numpy.ndarray
            Raster tiles as (tileSegments, tileImageData)

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadManager.startReadWorkers(self)
        Start the requested read workers, and set up the buffer they
        will feed into.

#### &nbsp;&nbsp;&nbsp;&nbsp; StatsReadManager.worker(self)
        Function running in each read worker

### class TiledStatsResult
      Result of tiled per-segment statistics
      
      Attributes
      ----------
        timings : pyshepseg.timinghooks.Timers
          Timings for various key parts of the per-segment stats calculation

#### &nbsp;&nbsp;&nbsp;&nbsp; TiledStatsResult.\__init__(self)
        Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def calcPerSegmentSpatialStatsRIOS(imgfile, imgbandnum, segfile, colNamesAndTypes, userFunc, userParam=None, concurrencyStyle=None, missingStatsValue=-9999, outFile=None, outFileIsZarr=False)
        This function is deprecated. Consider using calcPerSegmentSpatialStatsTiled
        with a readCfg instead.
        
        Similar to the :func:`calcPerSegmentStatsTiledRIOS` function 
        but allows the user to calculate spatial statistics on the data
        for each segment. This is done by recording the location and value
        for each pixel within the segment. Once all the pixels are found
        for a segment the ``userFunc`` is called with the following parameters:
        
        ::
        
            pts, imgNullVal, intArr, floatArr, userParam
            
        where ``pts`` is List of :class:`SegPoint` objects. If 2D Numpy tile is prefered the
        ``userFunc`` can call :func:`convertPtsInto2DArray`. ``intArray`` is a 
        1D numpy array which all the integer output values are to be put (in the 
        same order given in ``colNamesAndTypes``). ``floatArr`` is a 1D numpy array 
        which all the floating point output values are to be put (in the same order
        given in ``colNamesAndTypes``). These arrays are initialised with 
        ``missingStatsValue`` so the function can skip any that it doesn't have
        values for. ``userParam`` is the same value passed to 
        this function and needs to be a type understood by Numba. 
        
        This function uses RIOS to perform the reading so it works in
        a memory-efficient way. Also, the number of read workers can 
        be varied by passing an instance of rios.applier.ConcurrencyStyle
        which may be helpful when reading from data sources with high 
        latency (ie S3). RIOS timeouts can also be changed using this method.
        Note that only 0 compute workers is supported and 
        computeWorkerKind must be set to CW_NONE.   
        
        Parameters
        ----------
          imgfile : string
            Path to input file for collecting statistics from
          imgbandnum : int
            1-based index of the band number in imgfile to use for collecting stats
          segfile : str or gdal.Dataset
            Path to segmented file or an open GDAL Dataset. Will collect stats 
            in imgfile for each segment in this file.
          colNamesAndTypes : list of ``(colName, colType)`` tuples
            This defines the names, types and order of the output RAT columns.
            ``colName`` should be a string containing the name of the RAT column to be
            created and ``colType`` should be one of ``gdal.GFT_Integer`` or 
            ``gdal.GFT_Real`` and this controls the type of the column to be created.
            Note that the order of columns given in this parameter is important as
            this dicates the order of the ``intArray`` and ``floatArr`` parameters to 
            ``userFunc``.
          userFunc : a Numba function (ie decorated with @jit or @njit).
            See above for description
          userParam : anything that can be passed to a Numba function
            This includes: arrays, scalars and @jitclass decorated classes.
          concurrencyStyle : rios.applier.ConcurrencyStyle
            Concurrency parameters for RIOS
          missingStatsValue : int
            The value to fill in for segments that have no data.
          outFile : str
            Name of a separate output file in which to write RAT columns. If
            this is None, then columns are written back to segfile. If this
            is to be a GDAL file, it will be updated if it exists, or created
            using the KEA driver (so should have '.kea' extension). If
            outFileIsZarr if set to True, the output file will be a RatZarr file,
            and will either be created or updated as appropriate.
          outFileIsZarr : bool
            Set to True if the outFile should be written as RatZarr format.

### def calcPerSegmentSpatialStatsTiled(imgfile, imgbandnum, segfile, colNamesAndTypes, userFunc, userParam=None, missingStatsValue=-9999, outFile=None, outFileIsZarr=False, readCfg=None)
        Similar to the :func:`calcPerSegmentStatsTiled` function 
        but allows the user to calculate spatial statistics on the data
        for each segment. This is done by recording the location and value
        for each pixel within the segment. Once all the pixels are found
        for a segment the ``userFunc`` is called with the following parameters:
        
        ::
        
            pts, imgNullVal, intArr, floatArr, userParam
            
        where ``pts`` is List of :class:`SegPoint` objects. If 2D Numpy tile is prefered the
        ``userFunc`` can call :func:`convertPtsInto2DArray`. ``intArray`` is a 
        1D numpy array which all the integer output values are to be put (in the 
        same order given in ``colNamesAndTypes``). ``floatArr`` is a 1D numpy array 
        which all the floating point output values are to be put (in the same order
        given in ``colNamesAndTypes``). These arrays are initialised with 
        ``missingStatsValue`` so the function can skip any that it doesn't have
        values for. ``userParam`` is the same value passed to 
        this function and needs to be a type understood by Numba. 
        
        Parameters
        ----------
          imgfile : string
            Path to input file for collecting statistics from
          imgbandnum : int
            1-based index of the band number in imgfile to use for collecting stats
          segfile : str or gdal.Dataset
            Path to segmented file or an open GDAL Dataset. Will collect stats 
            in imgfile for each segment in this file.
          colNamesAndTypes : list of ``(colName, colType)`` tuples
            This defines the names, types and order of the output RAT columns.
            ``colName`` should be a string containing the name of the RAT column to be
            created and ``colType`` should be one of ``gdal.GFT_Integer`` or 
            ``gdal.GFT_Real`` and this controls the type of the column to be created.
            Note that the order of columns given in this parameter is important as
            this dicates the order of the ``intArray`` and ``floatArr`` parameters to 
            ``userFunc``.
          userFunc : a Numba function (ie decorated with @jit or @njit).
            See above for description
          userParam : anything that can be passed to a Numba function
            This includes: arrays, scalars and @jitclass decorated classes.
          missingStatsValue : int
            The value to fill in for segments that have no data.
          outFile : str
            Name of a separate output file in which to write RAT columns. If
            this is None, then columns are written back to segfile. If this
            is to be a GDAL file, it will be updated if it exists, or created
            using the KEA driver (so should have '.kea' extension). If
            outFileIsZarr if set to True, the output file will be a RatZarr file,
            and will either be created or updated as appropriate.
          outFileIsZarr : bool
            Set to True if the outFile should be written as RatZarr format.
          readCfg : StatsReadConfig
            Config for read manager, allowing multi-threaded reading.
            Default will run with no read workers.

### def calcPerSegmentSpatialStats_riosFunc(info, inputs, outputs, otherArgs)
        Called by RIOS from inside calcPerSegmentSpatialStatsRIOS. Do 
        accumulation of statistics

### def calcPerSegmentStatsRIOS(imgfile, imgbandnum, segfile, statsSelection, concurrencyStyle=None, missingStatsValue=-9999, outFile=None, outFileIsZarr=False)
        This function is deprecated. Consider using calcPerSegmentStatsTiled
        with a readCfg instead.
        
        Calculate selected per-segment statistics for the given band 
        of the imgfile, against the given segment raster file. 
        Calculated statistics are written to the segfile raster 
        attribute table (RAT), so this file format must support RATs. 
        
        This function uses RIOS to perform the reading so it works in
        a memory-efficient way. Also, the number of read workers can 
        be varied by passing an instance of rios.applier.ConcurrencyStyle
        which may be helpful when reading from data sources with high 
        latency (ie S3). RIOS timeouts can also be changed using this method.
        Note that only 0 compute workers is supported and 
        computeWorkerKind must be set to CW_NONE.   
        
        Parameters
        ----------
          imgfile : string
            Path to input file for collecting statistics from
          imgbandnum : int
            1-based index of the band number in imgfile to use for collecting stats
          segfile : str
            Path to segmented file. Will collect stats 
            in imgfile for each segment in this file.
          statsSelection : list of tuples.
            One tuple for each statistic to be included. Each tuple is either 2
            or 3 elements::
            
              (columnName, statName)
            
            or ::
            
              (columnName, statName, parameter)
            
            The columnName is a string, used to name the column in the 
            output RAT. 
            The statName is a string used to identify which statistic 
            is to be calculated. Available options are::
            
              'min', 'max', 'mean', 'stddev', 'median', 'mode', 'percentile', 'pixcount'
            
            The 'percentile' statistic requires the 3-element form, with 
            the 3rd element being the percentile to be calculated.
        
            For example::
            
              [('Band1_Mean', 'mean'),
               ('Band1_stdDev', 'stddev'),
               ('Band1_LQ', 'percentile', 25),
               ('Band1_UQ', 'percentile', 75)]
            
            would create 4 columns, for the per-segment mean and 
            standard deviation of the given band, and the lower and upper 
            quartiles, with corresponding column names.
        
            Any pixels that are set to the nodata value of imgfile (if set)
            are ignored in the stats calculations. If there are no pixels
            that aren't the nodata value then the value passed in as
            missingStatsValue is put into the RAT for the requested
            statistics.
            The 'pixcount' statName can be used to find the number of
            valid pixels (not nodata) that were used to calculate the statistics.
            
          concurrencyStyle : rios.applier.ConcurrencyStyle
            Concurrency parameters for RIOS
          missingStatsValue : int or float
            What to set for segments that have no valid pixels in imgile
          outFile : str
            Name of a separate output file in which to write RAT columns. If
            this is None, then columns are written back to segfile. If this
            is to be a GDAL file, it will be updated if it exists, or created
            using the KEA driver (so should have '.kea' extension). If
            outFileIsZarr if set to True, the output file will be a RatZarr file,
            and will either be created or updated as appropriate.
          outFileIsZarr : bool
            Set to True if the outFile should be written as RatZarr format.

### def calcPerSegmentStatsTiled(imgfile, imgbandnum, segfile, statsSelection, missingStatsValue=-9999, outFile=None, outFileIsZarr=False, readCfg=None)
        Calculate selected per-segment statistics for the given band 
        of the imgfile, against the given segment raster file. 
        Calculated statistics are written to the segfile raster 
        attribute table (RAT), so this file format must support RATs. 
        
        Calculations are carried out in a memory-efficient way, allowing 
        very large rasters to be processed. Raster data is handled in 
        small tiles, attribute table is handled in fixed-size chunks. 
        
        Parameters
        ----------
          imgfile : string
            Path to input file for collecting statistics from
          imgbandnum : int
            1-based index of the band number in imgfile to use for collecting stats
          segfile : str or gdal.Dataset
            Path to segmented file or an open GDAL dataset. Will collect stats 
            in imgfile for each segment in this file.
          statsSelection : list of tuples.
            One tuple for each statistic to be included. Each tuple is either 2
            or 3 elements::
            
              (columnName, statName)
            
            or ::
            
              (columnName, statName, parameter)
            
            The columnName is a string, used to name the column in the 
            output RAT. 
            The statName is a string used to identify which statistic 
            is to be calculated. Available options are::
            
              'min', 'max', 'mean', 'stddev', 'median', 'mode', 'percentile', 'pixcount'
            
            The 'percentile' statistic requires the 3-element form, with 
            the 3rd element being the percentile to be calculated.
        
            For example::
            
              [('Band1_Mean', 'mean'),
               ('Band1_stdDev', 'stddev'),
               ('Band1_LQ', 'percentile', 25),
               ('Band1_UQ', 'percentile', 75)]
            
            would create 4 columns, for the per-segment mean and 
            standard deviation of the given band, and the lower and upper 
            quartiles, with corresponding column names.
        
            Any pixels that are set to the nodata value of imgfile (if set)
            are ignored in the stats calculations. If there are no pixels
            that aren't the nodata value then the value passed in as
            missingStatsValue is put into the RAT for the requested
            statistics.
            The 'pixcount' statName can be used to find the number of
            valid pixels (not nodata) that were used to calculate the statistics.
        
          missingStatsValue : int or float
            What to set for segments that have no valid pixels in imgile
          outFile : str
            Name of a separate output file in which to write RAT columns. If
            this is None, then columns are written back to segfile. If this
            is to be a GDAL file, it will be updated if it exists, or created
            using the KEA driver (so should have '.kea' extension). If
            outFileIsZarr if set to True, the output file will be a RatZarr file,
            and will either be created or updated as appropriate.
          outFileIsZarr : bool
            Set to True if the outFile should be written as RatZarr format.
          readCfg : StatsReadConfig
            Config for read manager, allowing multi-threaded reading.
            Default will run with no read workers.

### def calcPerSegmentStats_riosFunc(info, inputs, outputs, otherArgs)
        Called by RIOS from inside calcPerSegmentStatsRIOS. Do 
        accumulation of statistics

### def copyRatCols(srcRat, destRat)
        Copy all columns from srcRat to dstRat.
        
        This is intended only for use copying from a temporary RAT file, which
        should only contain the columns to be copied. Use outside this could be
        dangerous.
        
        Copies each column in fixed-size blocks, so quite memory-efficient.
        
        Parameters
        ----------
           srcRat : str
             Name of temp KEA file with RAT to copy
           destRat : str or gdal.Dataset
             Name (or Dataset) of destination GDAL file to which RAT is copied

### def createNoDataDict()
        Create the dictionary that holds counts of nodata seen for each 
        segment. The key is the segId, value is the count of nodata seen 
        for that segment in the image data.
        
        Returns
        -------
         nodataDict : numba.typed.Dict 
           Dictionary of nodata counts for each segment
           

### def createPagedRat()
        Create the dictionary for the paged RAT. Each element is a page of
        the RAT, with entries for a range of segment IDs. The key is the 
        segment ID of the first entry in the page. 
        
        The returned dictionary is initially empty. 

### def createSegDict()
        Create the Dict of Dicts for handling per-segment histograms. 
        Each entry is a dictionary, and the key is a segment ID.
        Each dictionary within this is the per-segment histogram for
        a single segment. Each of its entries is for a single value from 
        the imagery, the key is the pixel value, and the dictionary value 
        is the number of times that pixel value appears in the segment. 
        
        Returns
        -------
         segDict : numba.typed.Dict 
           Dictionary of histograms used for calculating statistics.

### def createSegSpatialDataDict()
        Create a dictionary where the key is the segment ID and the 
        value is a List of :class:`SegPoint` objects.
        
        Returns
        -------
          segDict : numba.typed.Dict
            Dictionary containing lists of :class:`SegPoint` objects.

### def createStatColumns(statsSelection, openRat)
        Create requested statistic columns on the segmentation image RAT.
        Statistic columns are of type gdal.GFT_Real for mean and stddev, 
        and gdal.GFT_Integer for all other statistics. 
        
        Return the column indexes for all requested columns, in the same
        order. 
        
        Parameters
        ----------
          statsSelection : list of tuples
            Same as passed to :func:`calcPerSegmentStatsTiled`
          openRat : OpenRatContainer
            The file handle(s) for the RAT file
            
        Returns
        -------
          colIndexList : list of ints
            A list of the indexes of each of the requested new columns
            in the same order as statsSelection.

### def createUserColumnsSpatial(colNamesAndTypes, openRat)
        Used by :func:`calcPerSegmentSpatialStatsTiled` to create columns specified
        in the ``colNamesAndTypes`` structure. 
        
        Returns a tuple with number of integer columns, number of float columns
        and a statsSelection_fast array for use by :func:`calcStatsForCompletedSegsSpatial`.
        
        Parameters
        ----------
          colNamesAndTypes : list of (colName, colType) tuples
            Same as passed to :func:`calcPerSegmentSpatialStatsTiled`.
          openRat : OpenRatContainer
            The file handle(s) for the RAT file
        
            
        Returns
        -------
          numIntCols : int
            The number of new Integer columns
          numFloatCols : int
            The number of new Float columns
          statsSelection_fast : int ndarray of shape (numStats, 5)
            Allows quick access to the types of stats required

### def doImageChecks(segfile, imgfile, imgbandnum)
        Do the checks that the segment file and image file that is being used to 
        collect the stats actually align. We refuse to process the files if they
        don't as it is not clear how they should be made to line up - this is up
        to the user to get right. Also checks that imgfile is not a float image.
        
        Check that there is a null value set on imgfile, and that the segfile
        has a histogram.
        
        The two rasters are opened read-only, and closed again afterwards.
        
        Parameters
        ----------
          segfile : str or gdal.Dataset
            Path to segmentation file or an open GDAL dataset. 
          imgfile : string
            Path to input file for collecting statistics from
          imgbandnum : int
            1-based index of the band number in imgfile to use for collecting stats
        
        Returns
        -------
          imgNullVal : numba.int64
            The null value set in the imgdata raster
          segSize : ndarray
            The Histogram column of the segfile, i.e. the pixel counts of
            each segment

### def equalProjection(proj1, proj2)
        Returns True if the proj1 is the same as proj2
        
        Stolen from rios/pixelgrid.py
        
        Parameters
        ----------
          proj1 : string
            WKT string for the first projection
          proj2 : string
            WKT string for the second projection
            
        Returns
        -------
          equal : bool
            Whether the projections are equal or not

### def makeFastStatsSelection(colIndexList, statsSelection)
        Make a fast version of the statsSelection data structure, combined
        with the global column index numbers.
        
        Return a tuple of
        
            (statsSelection_fast, numIntCols, numFloatCols)
        
        The statsSelection_fast is a single array, of shape (numStats, 5). 
        The first index corresponds to the sequence in statsSelection. 
        The second index corresponds to the STATSEL_* values. 
        
        Everything is encoded as an integer value in a single numpy array, 
        suitable for fast access within numba njit-ed functions. 
        
        This is all a bit ugly and un-pythonic. Not sure if there is
        a better way. 
        
        Parameters
        ----------
          colIndexList : list of ints
            The column indexes for all requested columns 
          statsSelection : list of tuples
            See :func:`tilingstats.calcPerSegmentStatsTiled` for a complete 
            description of this parameter.
        
        Returns
        -------
          statsSelection_fast : int ndarray of shape (numStats, 5)
            The statsSelection_fast structure
          intCount : int
            Number of int columns
          floatCount : int
            Number of float columns

### def makeOutRatKea(outFile)
        Create a small KEA file to write a RAT into. Return a single object
        with all the open GDAL handles on it.
        
        Parameters
        ----------
          outFile : str
            Name of output KEA file
        
        Returns
        -------
          openRat : OpenRatContainer
            Holds all the open GDAL handles

### def openEverything(readCfg, outFile, outFileIsZarr, tileSize, numXtiles, numYtiles, imgfile, imgbandnum, segfile, timings)
        Open all the input and output files, ready for the stats routines to
        do their work.

### def writeCompletePages(pagedRat, openRat, statsSelection_fast)
        Check for completed pages, and write them to the attribute table.
        Remove them from the pagedRat after writing.
        
        Parameters
        ----------
          pagedRat : numba.typed.Dict
            The RAT as a paged data structure
          attrTbl : gdal.RasterAttributeTable
            The Raster Attribute Table object for the file
          statsSelection_fast : int ndarray of shape (numStats, 5)
            Allows quick access to the types of stats required

## Values
    HAVE_RATZARR = False
    HAVE_RIOS = False
    NOPARAM = 4294967295
    PTS_TYPE = instance.jitclass.SegPoint#7435068f8750<x:uint32,y:uint32,val:int64>
    RAT_PAGE_SIZE = 100000
    STATID_MAX = 1
    STATID_MEAN = 2
    STATID_MEDIAN = 4
    STATID_MIN = 0
    STATID_MODE = 5
    STATID_PERCENTILE = 6
    STATID_PIXCOUNT = 7
    STATID_STDDEV = 3
    STATSEL_COLARRAYINDEX = 3
    STATSEL_COLTYPE = 2
    STATSEL_GLOBALCOLINDEX = 0
    STATSEL_PARAM = 4
    STATSEL_STATID = 1
    STATSSELFAST_NULLVAL = 4294967295
    STAT_DTYPE_FLOAT = 1
    STAT_DTYPE_INT = 0
    SegPointSpec = [('x', uint32), ('y', uint32), ('val', int64)]
    accumulateSegDict = CPUDispatcher(<function accumulateSegDict at 0x7435068db880>)
    accumulateSegSpatial = CPUDispatcher(<function accumulateSegSpatial at 0x7435060da480>)
    calcStatsForCompletedSegs = CPUDispatcher(<function calcStatsForCompletedSegs at 0x7435068dba60>)
    calcStatsForCompletedSegsSpatial = CPUDispatcher(<function calcStatsForCompletedSegsSpatial at 0x74350601cfe0>)
    checkSegComplete = CPUDispatcher(<function checkSegComplete at 0x7435068db920>)
    checkSegCompleteSpatial = CPUDispatcher(<function checkSegCompleteSpatial at 0x7435060da660>)
    convertPtsInto2DArray = CPUDispatcher(<function convertPtsInto2DArray at 0x7435060bff60>)
    convertPtsInto2DMaskArray = CPUDispatcher(<function convertPtsInto2DMaskArray at 0x7435060bfe20>)
    getRatPageId = CPUDispatcher(<function getRatPageId at 0x74350601dee0>)
    getSortedKeysAndValuesForDict = CPUDispatcher(<function getSortedKeysAndValuesForDict at 0x7435068f4400>)
    numbaTypeForImageType = int64
    ratPageSpec = [('startSegId', uint32), ('intcols', Array(int64, 2, 'A', False, aligned=True)), ('floatcols', Array(float32, 2, 'A', False, aligned=True)), ('complete', Array(bool, 1, 'A', False, aligned=True))]
    ratzarr = None
    segIdNumbaType = uint32
    segStatsSpec = [('pixVals', Array(int64, 1, 'A', False, aligned=True)), ('counts', Array(uint32, 1, 'A', False, aligned=True)), ('pixCount', uint32), ('min', int64), ('max', int64), ('mean', float32), ('stddev', float32), ('median', int64), ('mode', int64), ('missingStatsValue', int64)]
    statIDdict = {'min': 0, 'max': 1, 'mean': 2, 'stddev': 3, 'median': 4, 'mode': 5, 'percentile': 6, 'pixcount': 7}
    userFuncMeanCoord = CPUDispatcher(<function userFuncMeanCoord at 0x7435068f4a40>)
    userFuncNumEdgePixels = CPUDispatcher(<function userFuncNumEdgePixels at 0x7435068f54e0>)
    userFuncVariogram = CPUDispatcher(<function userFuncVariogram at 0x7435068f44a0>)
