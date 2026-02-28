# rios.riostests.testfootprint
    Test the behaviour of the footprint type. Reads two input files
    which have different extents, and checks that each of the footprint types
    behaves as it should. Currently tests with images which are offset
    from each other, but in the same projection, making calculation of the
    correct answer fairly simple.

## Functions
### def checkOutputExtent(ramp1, ramp2, outimg, footprintType)
        Check that the extent of the output image is appropriate for the
        given footprint type and the input extents.
        
        Return True is extent is correct

### def doSomething(info, inputs, outputs)
        Do nothing of importance, just to write an output file

### def makeExtentTuple(info)
        Make a single tuple of the extent, to allow for quick comparisons
        of whole extents

### def run()
        Run the test

## Values
    OFFSET_2ND_IMAGE = 1000
    OFFSET_PIXELS = 100
    PIX = 10
    TESTNAME = 'TESTFOOTPRINT'
