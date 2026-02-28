# rios.riostests.teststats
    Test the calculation of statistics by rios.calcstats. 

## Classes
### class Stats

#### &nbsp;&nbsp;&nbsp;&nbsp; Stats.\_\_init\_\_(self, mean, stddev, minval, maxval, median, mode)
        Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def calcMode(a, axis=0)
        Copied directly from scipy.stats.mode(), so as not to have a dependency on scipy. 

### def checkHistogram(band, imgArr, nullVal, iterationName)
        Do simple check(s) on the histogram

### def compareStats(stats1, stats2, iterationName, relativeTolerance)
        Compare two Stats instances, and report differences. Also
        return True if all OK. 

### def doAllNull(info, inputs, outputs, otherargs)
        Called from RIOS. Write an output which is all nulls

### def doit(info, inputs, outputs, otherargs)
        Called from RIOS.
        
        Re-write the input, with scaling and change of datatype

### def equalTol(a, b, tol)
        Compare two values to within a tolerance. If the difference
        between the two values is smaller than the tolerance, 
        then return True

### def getStatsFloatVal(band, metadataName)
        Get a single float value from the band metadata, or None if
        it is not present

### def getStatsFromArray(arr, nullVal)
        Work out the statistics directly from the image array. 
        Return a Stats instance

### def getStatsFromBand(band)
        Get statistics from given band object, return Stats instance

### def run()
        Run a test of statistics calculation

### def runOneTest(driverName, creationOptions, fileDtype, scalefactor, offset, rampInfile, ext, omit, singlePass, thematic, noNull=False)
        Run a full test of stats and histogram for the given configuration

### def testAllNull()
        Write an output file which is all nulls, and check that the stats and
        histogram behave appropriately.

### def testForDriverAndType(driverName, creationOptions, fileDtype, scalefactor, offset, rampInfile, ext)
        Run a set of stats tests for the given drive and datatype.

## Values
    TESTNAME = 'TESTSTATS'
    floatGDALTypes = (6, 7)
    hugeIntGDALTypes = (5, 4, 13, 12)
