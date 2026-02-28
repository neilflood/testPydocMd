# rios.riostests.testratapplier
    Test the ratapplier functionality

## Functions
### def makeTestFile(imgfile, withRat=True, nRatRows=100, gdalType=1, driverName='HFA')

### def myFunc(info, inputs, outputs)

### def myFuncCopyCol(info, inputs, outputs)

### def myFuncDiffFile(info, inputs, outputs)

### def myFuncNewRat(info, inputs, outputs)

### def myFuncZarrFile(info, inputs, outputs)

### def myStringDTypeFunc(info, inputs, outputs)

### def myStringDTypeReadFunc(info, inputs, outputs, otherArgs)

### def rasterReduceFunc(info, inputs, outputs)

### def ratReduceFunc(info, inputs, outputs)

### def run()
        Run tests of the ratapplier

### def testDifferentOutput(imgfile, imgfile2)

### def testNewRat(imgfile4)

### def testOutputSameFile(imgfile)

### def testReduceRat(imgfile, imgfile3)
        This test creates a new output image, with all odd pixel values 
        replaced with the even number above it. The RAT must then be copied across
        with the same reduction performed. In this case, only the even numbered 
        rows are written

### def testStringDType(imgfile)
        Test the use of GFT_String columns with numpy's StringDType

### def testZarrInputOutput(zarrfile)
        Test reading and writing in the same Zarr file

### def testZarrOutput(imgfile, zarrfile)
        Test output to a Zarr RAT

## Values
    TESTNAME = 'TESTRATAPPLIER'
    ratzarr = None
