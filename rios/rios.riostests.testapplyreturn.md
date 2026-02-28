# rios.riostests.testapplyreturn
    Test the operation of the apply() return object, running with
    multiple compute threads.

## Functions
### def calcAverage(file1, cwKind)
        Use RIOS to calculate the average value over the file.

### def checkResult(avg_threads, avg_subproc)
        Read in from the given file, and check that it matches what we 
        think it should be

### def doSums(info, inputs, outputs, otherargs)
        Called from RIOS.
        
        Calculate the sum and count of all pixels in each block. 

### def run()
        Run the test

## Values
    TESTNAME = 'TESTAPPLYRETURN'
