# pyshepseg.cmdline.run_seg
    Testing harness for pyshepseg. Handy for running a basic segmentation
    but it is suggested that users call the module directly from a Python
    script and handle things like scaling the data in an appripriate
    manner for their application.

## Functions
### def getCmdargs()
        Get the command line arguments.

### def main()

### def readImageBands(cmdargs)
        Read in the requested bands of the given image. Return
        a tuple of the image array and the null value.

### def writeClusterCentresToMetadata(bandObj, km)
        Pulls out the cluster centres from the kmeans object 
        and writes them to the metadata (under CLUSTER_CNTRS_METADATA_NAME)
        for the given band object.

### def writeOutput(cmdargs, seg, segSize, spectSum, kmeansObj)
        Write the segmentation to an output image file. Includes a 
        colour table

## Values
    CLUSTER_CNTRS_METADATA_NAME = pyshepseg_cluster_cntrs
    DFLT_MAX_SPECTRAL_DIFF = auto
    DFLT_OUTPUT_DRIVER = KEA
    GDAL_DRIVER_CREATION_OPTIONS = {'KEA': [], 'HFA': ['COMPRESS=YES']}
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
