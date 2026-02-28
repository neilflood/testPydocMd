# rios.riostests.testreaderinfo
    Test the ReaderInfo object for correctness

## Functions
### def checkLookupFunctions(info, inputs, outputs, otherargs)
        Check the array-based lookup functions of the info object

### def checkReaderInfo(info, inputs, outputs, otherargs)
        For each block, check that the info object corresponds to the
        row/col taken from the input file

### def run()
        Run the test

## Values
    TESTNAME = 'TESTREADERINFO'
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
