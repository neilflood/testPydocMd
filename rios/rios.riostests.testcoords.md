# rios.riostests.testcoords
    A simple set of tests of the access to coordinates from with RIOS. 
    There are a number of routines on the ReaderInfo class which
    allow the RIOS use function to access coordinate systems of the 
    working grid, and the underlying rasters, and this tests whether 
    these behave as expected, in a range of circumstances. 

## Functions
### def checkCoordList(list1, list2)
        Check equality of given lists of (x,y) coords. Return True if they
        are the same, False otherwise. 

### def checkCoords(testConditionStr, otherargs, tstPixList, centresList)
        Check that the coordinate lists created in otherargs are the same as 
        the correct answers passed in to this routine. 

### def getCoords(info, inputs, outputs, otherargs)
        Accumulate some lists of the coordinates of the top-left corner pixel
        of each block, expressed in a variety of ways

### def run()
        Run the test

## Values
    CENTRES_1FILE_NOOVERLAP = [(500005.0, 6999995.0), (502565.0, 6999995.0), (500005.0, 6997435.0), (502565.0, 6997435.0)]
    CENTRES_1FILE_OVERLAP2 = [(499985.0, 7000015.0), (502545.0, 7000015.0), (499985.0, 6997455.0), (502545.0, 6997455.0)]
    CENTRES_2FILE_OVERLAP2 = [(500985.0, 6999015.0), (503545.0, 6999015.0), (500985.0, 6996455.0), (503545.0, 6996455.0)]
    HALFPIX = 5.0
    MARGIN = 20
    NUMBLOCKSX = 2
    NUMBLOCKSX_WITHOFFSET = 2
    NUMBLOCKSY = 2
    NUMBLOCKSY_WITHOFFSET = 2
    OFFSET_2ND_IMAGE = 1000
    OFFSET_PIXELS = 100
    OLAP = 2
    PIX = 10
    STEP = 2560
    TESTNAME = 'TESTCOORDS'
    TSTPIX_1FILE_NOOVERLAP = [(110, 110)]
    TSTPIX_1FILE_OVERLAP2 = [(112, 112)]
    TSTPIX_2FILE_OVERLAP2 = [(12, 12)]
    TSTPT_X = 501105.0
    TSTPT_Y = 6998895.0
    XSTART = 500000.0
    YSTART = 7000000.0
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
