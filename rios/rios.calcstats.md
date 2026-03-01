# rios.calcstats
    This module creates pyramid layers and calculates statistics for image
    files. Much of it was originally for ERDAS Imagine files but should work
    with any other format that supports pyramid layers and statistics

## Classes
### class HistogramParams
      Work out the various parameters needed by GDAL to compute a histogram.
      The inferences are based on the pixel datatype. Some (but not all) of the
      parameters are also used when doing a single-pass histogram.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HistogramParams.\_\_init\_\_(self, band, minval, maxval)
          Initialize self.  See help(type(self)) for accurate signature.

### class ProgressUserData

### class SinglePassAccumulator
      Accumulator for statistics and histogram for a single band. Used when
      doing single-pass stats and/or histogram.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.\_\_init\_\_(self, includeStats, includeHist, dtype, nullval, thematic)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.addTwoHistograms(hist1, hist2)
          Add the two given histograms together, and return the result.
          
          If one is longer than the other, the shorter one is added to it.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.doHistAccum(self, arr)
          Accumulate the histogram with counts from the given arr. For signed
          int types, maintain two separate count arrays, one for positive
          values and one for negatives. This is due to using numpy.bincount()
          to do the counting.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.doStatsAccum(self, arr)
          Accumulate basic stats for the given array

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.finalStats(self)
          Return the final values of the four basic statistics
          (minval, maxval, mean, stddev)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.fullHist(self)
          Return the full histogram, as (minval, maxval, counts)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.removeNullFromCounts(counts, nullval)
          The counts will include a count for the null value. Set this to zero,
          and if it is at the end of the count array, truncate this back to
          the next biggest non-zero count.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassAccumulator.updateHist(self, newCounts, positive)
          Update the current histogram counts. If positive is True, then
          the counts for positive values are updated, otherwise those for the
          negative values are updated.

### class SinglePassManager
      The required info for dealing with single-pass pyramids/statistics/histogram.
      There is some complexity here, because the decisions about what to do are
      a result of a number of different factors. We attempt to make these decisions
      as early as possible, and store the decisions on this object, so they can
      just be checked later.
      
      The general intention is that wherever possible, the pyramids, basic
      statistics, and histogram, will all be done with the single-pass methods.
      When this is not possible, or has been explicitly disabled, then it will
      fall back to using GDAL's methods, after the whole raster has been written.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassManager.\_\_init\_\_(self, outfiles, controls, workinggrid, tmpfileMgr)
          Check whether single-pass is appropriate and/or supported for
          all output files.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassManager.checkDriverPyramidSupport(self, outfiles, controls, tmpfileMgr)
          For all the format drivers being used for output, check whether they
          support direct writing of pyramid layers. Return a dictionary keyed
          by driver name, with boolean values.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassManager.doSinglePassHistogram(self, symbolicName)
          Return True if we should do single-pass histogram, False
          otherwise, based on what has been requested, the datatype of
          the raster.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassManager.doSinglePassPyramids(self, symbolicName)
          Return True if we should do single-pass pyramids layers, False
          otherwise. Decision depends on choices for omitPyramids,
          singlePassPyramids, and overviewAggType.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassManager.doSinglePassStatistics(self, symbolicName)
          Return True if we should do single-pass basic statistics, False
          otherwise.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SinglePassManager.initFor(self, ds, symbolicName, seqNum, arr)
          Initialise for the given output file

## Functions
### def addBasicStatsGDAL(ds, approx_ok)
        Add basic statistics (min, max, mean, stddev) to all bands of the
        given Dataset, using GDAL's function. If approx_ok is True, then
        much faster approximate statistics will be calculated (in particular,
        the min and max will only be approximate).
        
        Assumes that any desired null value has already been set on each
        band of the Dataset.
        
        Return a list of the minimum and maximum values for each band, in case
        this is required later for the histogram.

### def addHistogramsGDAL(ds, minMaxList, approx_ok)
        Add histograms to all bands of the given Dataset, using GDAL's own
        function. If approx_ok is True, then much faster approximate histograms
        will be calculated (i.e. the pixel counts will be in proportion, but
        not exactly accurate).
        
        Assumes that any desired null value has already been set on each
        band of the Dataset.
        
        The minMaxList is as returned by addBasicStatsGDAL.

### def addPyramid(ds, progress, minoverviewdim=128, levels=[4, 8, 16, 32, 64, 128, 256, 512], aggregationType=None)
        Adds Pyramid layers to the dataset. Adds levels until
        the raster dimension of the overview layer is < minoverviewdim,
        up to a maximum level controlled by the levels parameter. 
        
        Assumes that any desired null value has already been set on each
        band of the Dataset.
        
        Uses gdal.Dataset.BuildOverviews() to do the work. 

### def addStatistics(ds, progress, ignore=None, approx_ok=False)
        Calculates statistics and adds them to the image. As of version
        2.0.5, this function is no longer used directly with RIOS, and
        is maintained purely for backward compatibility with programs
        which call it directly.
        
        Uses gdal.Band.ComputeStatistics() for mean, stddev, min and max,
        and gdal.Band.GetHistogram() to do histogram calculation. 
        The median and mode are estimated using the histogram, and so 
        for larger datatypes, they will be approximate only. 
        
        For thematic layers, the histogram is calculated with as many bins 
        as required, for athematic integer and float types, a maximum
        of 256 bins is used.
        
        Note that this routine will use the given ignore value to set the
        no-data value (i.e. null value) on the dataset, using the same value
        for every band.
        
        Obsolete from version 2.0.5.
        See addBasicStatsGDAL() and addHistogramsGDAL() for replacements.

### def calcStats(ds, progress=None, ignore=None, minoverviewdim=128, levels=[4, 8, 16, 32, 64, 128, 256, 512], aggregationType=None, approx_ok=False)
        This function is no longer used internally, and is maintained
        purely for backward compatibility for programs which called it
        directly.
        
        Calls the addPyramid and addStatistics functions, to add pyramid
        layers (i.e. overviews), and basic statistics and histogram,
        to the given open Dataset ``ds``. See the docstrings for those
        functions for details.

### def computeStatsGDAL(band, approx_ok)
        Compute basic statistics of a single band, using GDAL's function.
        Returns the values as a tuple (does NOT write anything into the file).
        
        If there are no non-null pixels, then all stats are returned as None.
        
        Returns (minval, maxval, mean, stddev)

### def findOrCreateColumn(ratObj, usage, name, dtype)
        Returns the index of an existing column matched
        on usage. Creates it if not already existing using 
        the supplied name and dtype
        Returns a tuple with index and a boolean specifying if 
        it is a new column or not

### def finishSinglePassHistogram(ds, singlePassMgr, symbolicName, seqNum)
        Finish the single-pass histogram, and write to file. Also writes the median
        and mode, which are estimated from the histogram.

### def finishSinglePassStats(ds, singlePassMgr, symbolicName, seqNum)
        Finish the single-pass basic statistics for all bands of the given
        file, and write them into the file.

### def handleSinglePassActions(ds, arr, singlePassMgr, symbolicName, seqNum, xOff, yOff, timings)
        Called from writeBlock, to handle any single-pass actions which may
        be required.

### def linearHistFromDirect(desiredNbins, step, counts)
        Take a direct-binFunction histogram and re-bin it to create a
        linear-binFunction equivalent. This is intended for use with counts
        created with the single-pass algorithm, for the cases when we would
        otherwise have chosen a linear-binFunction histogram.
        Generally this is to save writing a very large number of counts.
        
        The minval and maxval will be preserved. The given desiredNbins is the
        number of bins desired in the new histogram. The counts are the old
        counts, and will be re-calculated with the requested number of bins.
        
        We preserve the total count, so the new histogram refers to the same
        total number of pixels.

### def progressFunc(value, string, userdata)
        Progress callback for BuildOverviews

### def setNullValue(ds, nullValue)
        Set the given null value on all bands of the given Dataset

### def writeBasicStats(band, minval, maxval, meanval, stddev, approx_ok)
        Write the given basic statistics into the given band.
        
        It is assumed that by this point, we have set the null value on the band
        (this is normally done when the file is opened).

### def writeBlockPyramids(ds, arr, singlePassMgr, symbolicName, xOff, yOff)
        Calculate and write out the pyramid layers for all bands of the block
        given as arr. Called when doing single-pass pyramid layers.

### def writeHistogram(ds, band, hist, histParams)
        Write the given histogram into the band object. Also use the histogram
        to estimate median and mode, and write them as well.

## Values
    DEFAULT_MINOVERVIEWDIM = 128
    DEFAULT_OVERVIEWAGGREGRATIONTYPE = 'NEAREST'
    DEFAULT_OVERVIEWLEVELS = [4, 8, 16, 32, 64, 128, 256, 512]
    dfltOverviewLvls = None
    gdalFloatTypes = {6, 7}
    gdalLargeIntTypes = {2, 3, 4, 5, 12, 13, 14}
    numpySignedIntTypes = (<class 'numpy.int8'>, <class 'numpy.int16'>, <class 'numpy.int32'>, <class 'numpy.int64'>)
    numpyUnsignedIntTypes = (<class 'numpy.uint8'>, <class 'numpy.uint16'>, <class 'numpy.uint32'>, <class 'numpy.uint64'>)
