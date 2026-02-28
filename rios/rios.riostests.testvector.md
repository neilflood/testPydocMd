# rios.riostests.testvector
    A simple test of the vector inputs. 
    
    Creates a simple raster, and a simple vector, using straight gdal/ogr.
    Then reads the raster, masked by the vector, and calculates the mean of the masked
    area. Does the same thing with straight numpy, and checks the results. 

## Functions
### def calcMeanWithNumpy()
        Calculate the mean using numpy. This kind of relies on just "knowing" how the 
        vector was generated. It would be better if the part that generated the vector
        had returned a bit more information that I could just use here, but didn't get
        too carried away. Should tidy it up, though. 

### def calcMeanWithRiosApplier(imgfile, vecfile)
        Use RIOS's vector facilities, through the applier, to calculate the
        mean of the image within the vector

### def meanWithinVec(info, inputs, outputs, otherargs)
        Called from RIOS applier to accumulate sum and count of values
        within a vector

### def nearlyEqual(a, b, tol=0.0001)
        Return True if the two values are equal to within the given tolerance

### def run()
        Run the simple vector test

## Values
    TESTNAME = 'TESTVECTOR'
