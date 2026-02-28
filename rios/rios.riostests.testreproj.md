# rios.riostests.testreproj
    Test reprojection of input file.
    
    Reprojects an input file using GDAL directly, then reads these two files as
    RIOS inputs, allowing RIOS to reproject the original on-the-fly. They should
    thus be the same when checked inside the userFunction. The check is a per-pixel
    match, looking for zero mis-matched pixel values.
    
    This test is a little bit sensitive, because the code in GDAL is not completely
    stable. With this in mind, don't be in a hurry to change the detail of this test.

## Functions
### def checkMatch(ramp1, ramp2, resample)
        Use RIOS to check that the files are the same. Ramp1 will be reprojected
        on-the-fly by RIOS, to match ramp2.

### def checkNegativeNull()
        Check that reprojection still works when the null value is negative. There
        was a bug introduced in GDAL 3.9.0, and fixed in 3.9.2, which meant this
        could fail under some circumstances. We have a workaround, in imagereader.py,
        and this test is in place to check that workaround.

### def doNeg(info, inputs, outputs, otherargs)
        Reads input with a reprojection, and check null handling

### def makeNewPixgrid(filename)
        Make a pixgrid for the reprojected version of the input file

### def reprojFile(ramp1, ramp2, resampleMethod, nullval)
        Use gdalwarp to reproject the file, independently of RIOS. Does a
        bit of work to ensure that the extent of the reprojected file is aligned
        to nice neat numbers, in a way which matches what RIOS will probably do

### def run()
        Run the test

### def userFunc(info, inputs, outputs, otherargs)
        Check pixel-by-pixel match

## Values
    TESTNAME = 'TESTREPROJECTION'
