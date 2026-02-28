# pyshepseg.cmdline.subset
    Test harness for subsetting a segmented image

## Functions
### def getCmdargs()
        Get the command line arguments.

### def getExtentOfMaskForInfile(infile, maskfile)
        Get the extent of maskfile in the pixel coordinates of infile.
        Returns (tlx, tly, xsize, ysize)

### def getPixelCoords(fname, coords)
        Open the supplied file and work out what coords (ulx, uly, lrx, lry)
        are in pixel coordinates and return (tlx, tly, xsize, ysize)

### def main()

## Values
    DFLT_OUTPUT_DRIVER = KEA
    GDAL_DRIVER_CREATION_OPTIONS = {'KEA': [], 'HFA': ['COMPRESS=YES']}
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
