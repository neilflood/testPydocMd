# rios.riostests.testavg
    Does a broad, general test of the applier functionality. 
    
    Generates a pair of images, and then applies a function to calculate
    the average of them. Checks the resulting output against a known 
    correct answer. 
    
    Prints a message stderr if something wrong.  

## Functions
### def calcAverage(file1, file2, avgfile)
        Use RIOS to calculate the average of two files.

### def checkResult(avgfile)
        Read in from the given file, and check that it matches what we 
        think it should be

### def doAvg(info, inputs, outputs)
        Called from RIOS.
        
        Calculate the average of the input files. 

### def run()
        Run the test

## Values
    TESTNAME = 'TESTAVG'
