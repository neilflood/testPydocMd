# rios.riostests.testavgthreads
    Does a basic test of rios concurrency, using the CW_THREADS option.
    
    Generates a pair of images, and then applies a function to calculate
    the average of them. Checks the resulting output against a known 
    correct answer. 
    
    Steals heavily from testavg

## Functions
### def calcAverage(file1, file2, avgfile)
        Use RIOS to calculate the average of two files using MPI.

### def checkResult(avgfile)
        Read in from the given file, and check that it matches what we 
        think it should be

### def doAvg(info, inputs, outputs)
        Called from RIOS.
        
        Calculate the average of the input files. 

### def run()
        Run the test

## Values
    TESTNAME = 'TESTAVGTHREADS'
    TEST_NCPUS = 2
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
