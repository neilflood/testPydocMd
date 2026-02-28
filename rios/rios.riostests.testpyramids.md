# rios.riostests.testpyramids
    Test that pyramid layers (i.e overviews) are written correctly, in both the
    single-pass and GDAL cases

## Functions
### def checkPyramids(filename, singlePass)
        Check that the pyramids as written correspond to the base raster
        in the file. If singlePass is True, then we should have centred the low-res
        pixels, if it is False, then GDAL calculated them, and it appears not to
        centre the low-res pixels (which I believe to be a bug).

### def doit(info, inputs, outputs)

### def run()
        Run tests of pyramid layers (i.e. overviews)

## Values
    TESTNAME = 'TESTPYRAMIDS'
