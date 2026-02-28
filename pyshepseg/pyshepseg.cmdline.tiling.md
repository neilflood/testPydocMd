# pyshepseg.cmdline.tiling
    Testing harness for running pyshepseg in the tiling mode. 
    Handy for running a basic segmentation
    but it is suggested that users call the module directly from a Python
    script and handle things like scaling the data in an appripriate
    manner for their application.

## Functions
### def doPerSegmentStats(cmdargs)
        If requested, calculate RAT columns of per-segment statistics

### def getCmdargs()
        Get the command line arguments.

### def main()

## Values
    CLUSTER_CNTRS_METADATA_NAME = pyshepseg_cluster_cntrs
    DFLT_MAX_SPECTRAL_DIFF = auto
    DFLT_OUTPUT_DRIVER = KEA
    GDAL_DRIVER_CREATION_OPTIONS = {'KEA': [], 'HFA': ['COMPRESS=YES']}
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
