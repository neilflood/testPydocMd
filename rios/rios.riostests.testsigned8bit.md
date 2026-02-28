# rios.riostests.testsigned8bit
    Test the use of signed 8-bit rasters. The Int8 type was added to GDAL
    in version 3.7, and only later supported properly by RIOS.
    
    This does not test the earlier GDAL implementation using 'PIXELTYPE=SIGNEDBYTE',
    as this is not supported by RIOS.

## Functions
### def checkHistogram(outimg, ncols)
        Check that the histogram on the output file is correct

### def copy(info, inputs, outputs, otherargs)
        Called from RIOS. Write input to output

### def readAndWrite(inimg, outimg, nRows, nCols)
        Create a small signed 8-bit raster, and use RIOS to read it and write it
        out again, testing both the reading and writing

### def run()
        Run a test of statistics calculation

## Values
    TESTNAME = 'TESTSIGNED8BIT'
