# rios.riostests.testsetinputnull
    Test the controls.setInputNoDataValue() function.

## Functions
### def checkNulls(info, inputs, outputs, otherargs)
        Check the behaviour of nulls on each of the two input files.
        
        The working grid is in a different projecrtion to the input files,
        so the resampling of the inputs will have resampled across the boundary
        between null and non-null areas of the images. The input files do not
        have a null value set, but the img2 file has had an over-ride null
        set via controls.setInputNoDataValue. So, the behaviour should be
        different. The img1 file should have some smeared values present, while
        img2 should not.
        
        We also test that this is honoured by info.getNoDataValueFor

### def run()
        Run the test

## Values
    TESTNAME = 'TESTSETINPUTNULL'
