# pyshepseg.utils
    Utility functions for working with segmented data.

## Classes
### class WorkerErrorRecord
      Hold a record of an exception raised in a remote worker.

#### &nbsp;&nbsp;&nbsp;&nbsp; WorkerErrorRecord.\__init__(self, exc, workerType)
        Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def addOverviews(ds)
        Add raster overviews to the given file.
        Mimic rios.calcstats behaviour to decide how many overviews.
        
        Parameters
        ----------
          ds : gdal.Dataset
            Open Dataset for the raster file

### def deprecationWarning(msg, stacklevel=2)
        Print a deprecation warning to stderr. Includes the filename
        and line number of the call to the function which called this.
        The stacklevel argument controls how many stack levels above this
        gives the line number.
        
        Implemented in mimcry of warnings.warn(), which seems very flaky.
        Sometimes it prints, and sometimes not, unless PYTHONWARNINGS is set
        (or -W is used). This function at least seems to work consistently.

### def estimateStatsFromHisto(bandObj, hist)
        As a shortcut to calculating stats with GDAL, use the histogram 
        that we already have from calculating the RAT and calc the stats
        from that. 

### def formatTimingRpt(summaryDict)
        Format a report on timings, given the output of Timers.makeSummaryDict()
        Example usage::
        
            tiledSegResult = doTiledShepherdSegmentation(...)
            timings = tiledSegResult.timings
            summaryDict = timings.makeSummaryDict()
            reportStr = formatTimingRpt(summaryDict)
            print(reportStr)
        
        Return a single string of the formatted report.

### def reportWorkerException(exceptionRecord)
        Report the given WorkerExceptionRecord object to stderr

### def writeColorTableFromRatColumns(segfile, redColName, greenColName, blueColName)
        Use the values in the given columns in the raster attribute 
        table (RAT) to create corresponding color table columns, so that 
        the segmented image will display similarly to same bands of the 
        the original image. 
        
        The general idea is that the given columns would be the per-segment 
        mean values of the desired bands (see tiling.calcPerSegmentStatsTiled()
        to create such columns). 
        
        Parameters
        ----------
          segfile : str or gdal.Dataset
            Filename of the completed segmentation image, with RAT columns
            already written.  Can be either the file name string, or
            an open Dataset object.
          redColName : str
            Name of the column in the RAT to use for the red color
          greenColName : str
            Name of the column in the RAT to use for the green color
          blueColName : str
            Name of the column in the RAT to use for the blue color

### def writeRandomColourTable(outBand, nRows)
        Attach a randomly-generated colour table to the given segmentation
        image. Mainly useful so the segmentation boundaries can be viewed,
        without any regard to the meaning of the segments.
        
        Parameters
        ----------
          outBand : gdal.Band
            Open GDAL Band object for the segmentation image
          nRows : int
            Number of rows in the attribute table, equal to the
            number of segments + 1.

## Values
    DEFAULT_MINOVERVIEWDIM = 100
    DEFAULT_OVERVIEWLEVELS = [4, 8, 16, 32, 64, 128, 256, 512]
    deprecationAlreadyWarned = set()
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    gdalFloatTypes = {6, 7}
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
