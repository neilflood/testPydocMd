# rios.riostests.testoverlap
    Full test of the overlap mechanism.
    
    Apply a focal maximum filter to a ramp input, and check that RIOS's
    output gives the same as applying it to the array in memory. If no
    overlap is set, then it will find a few hundred incorrect pixles,
    but with the right overlap, there should be none.

## Functions
### def doFilter(info, inputs, outputs, otherargs)
        Apply median filter to input

### def readArr(filename)
        Read the whole of first band of given file

### def run()
        Run the test

## Values
    TESTNAME = 'TESTOVERLAP'
