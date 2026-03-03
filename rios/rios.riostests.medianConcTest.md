# rios.riostests.medianConcTest
    A simple stand-alone program to use for testing concurrency. This is NOT
    a part of the standard test suite, but just for use during development.

## Functions
### def doMedian(info, inputs, outputs, otherargs)
        Calculate per-pixel median of the list of files. Sam's code

### def getCentroid(tile)
        Query the shapefile of centroids to

### def getCmdargs()
        Get command line arguments

### def main()

### def makeRefPixgrid(img)

### def numbaMedian(data, nodata)
        Function utilises Numba to calculate median from stacked arrays.
        returns 3d array. Sam's code.

### def searchStac(cmdargs)
        Search the STAC server for suitable tiles. Return a dictionary
        of tiles, keyed by date.

## Values
    CW_AWSBATCH = 'CW_AWSBATCH'
    CW_NONE = 'CW_NONE'
    CW_THREADS = 'CW_THREADS'
    RIOS_VERSION = '2.0.9'
    collection = 'sentinel-2-l2a'
    stacServer = 'https://earth-search.aws.element84.com/v1/'
