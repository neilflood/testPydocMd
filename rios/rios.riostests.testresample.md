# rios.riostests.testresample
    A simple test with resampling required. Two input images
    are generated on a slightly different grid, and then rios averages
    the two, allowing a resample. 

## Functions
### def calcAverage(file1, file2, avgfile)
        Use RIOS to calculate the average of two files. Allows
        nearest-neighbour resampling of the second file. 

### def checkResult(avgfile)
        Read in from the given file, and check that it matches what we 
        think it should be

### def doAvg(info, inputs, outputs)
        Called from RIOS.
        
        Calculate the average of the input files. 

### def run()
        Run the test

## Values
    TESTNAME = 'TESTRESAMPLE'
