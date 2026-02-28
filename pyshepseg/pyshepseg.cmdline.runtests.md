# pyshepseg.cmdline.runtests
    Run tests of the pyshepseg package. Use a generated dataset, sufficient 
    to allow a meaningful segmentation. Note that the generated dataset is not 
    sufficiently complex to be a strong test of the Shepherd segmentation
    algorithm, but merely whether this implementation of the algorithm is
    coded to behave sensibly. 

## Functions
### def checkRatColumns(segfile, ratfile, fileIsRatZarr, allStatsCols)
        Check that the contents of the stats columns is the same in the segfile
        and the given rat file. The segfile is assumed to be GDAL-based,
        but the ratfile could be RatZarr.

### def checkSegmentation(imgfile, segfile, meanColNames, stdColNames)
        Check whether the given segmentation of the given image file 
        is "correct", by some measure(s). 

### def checkSpatialColumns(segfile, eastingCol, northingCol)
        Do a quick check of eastingCol and northingCol which were
        calculated using calcPerSegmentSpatialStatsTiled().
        
        Returns True if the calculated coordinates are the same
        as the actual coordinates (within a tolerance).

### def checkSubset(outsegfile, subset_segfile)
        Check the behavour of tiling.subsetImage()
        by doing a subset and checking that the new values
        can successfully be translated to the old using the
        origSegIdColName parameter.

### def createMultispectral(truesegfile, outfile)
        Reads the given true segment file, and generates a multi-spectral
        image which should segment in a similar way. 

### def createPallete(numSeg)
        Return a "pallete" of 3-band colours, one for each segment. 
        
        The colours are just made up, and have no particular meaning. The
        main criterion is that they be distinct, sufficiently so that 
        two adjacent segments should always come out in different colours. 
        
        Return value is an array of shape (numSeg, 3). Individual colour 
        values are in the range [0, 10000], and so the array has type uint16. 
        
        Note that the index into this array would be (segmentID - 1). 

### def generateTrueSegments(outfile)
        This routine generates the true segments from the initial segment
        centres hardwired in the initialCentres variable. 
        
        Each pixel is in the segment for its closest centre coordinate. 
        
        Saves the generated segment layer into the given raster filename,
        with format KEA. 

### def getCmdargs()
        Get command line arguments

### def main()
        Main routine

### def makeRATcolumns(outsegfile, imagefile, outFile=None, outFileIsZarr=False, useRIOS=False, numReadWorkers=0)
        Add some columns to the RAT, with useful per-segment statistics

### def makeSpatialRATColumns(segfile, imagefile, outFile=None, outFileIsZarr=False, useRIOS=False, numReadWorkers=0)
        Create some RAT columns for checking the spatial stats 
        functionality. 
        
        Here we use the tilingstats.userFuncMeanCoord function passed to
        calcPerSegmentSpatialStatsTiled so that the mean eastings and northings
        are calculated. These can be easily checked later on.

### def readColumn(ratfile, colName, fileIsRatZarr)
        Read the given column from the given RAT file. Copes with either
        a GDAL-based RAT or a RatZarr file, determined by the fileIsRatZarr
        parameter.
        
        Return an array of the column values. 

### def readSeg(segfile, xoff=0, yoff=0, win_xsize=None, win_ysize=None)
        Open and read the given segfile. Return an image array of the 
        segment ID values

### def vecPcntDiff(v1, v2)
        Calculate a percentage difference between two vectors

## Values
    ConcurrencyStyle = None
    HAVE_RIOS = False
    NBANDS = 3
    initialCentres = [(116, 3495), (142, 3100), (236, 6033), (290, 796), (297, 6152), (310, 5318), (409, 5867), (410, 2125), (442, 2913), (472, 1135), (486, 5296), (628, 667), (655, 2677), (672, 4001), (677, 5513), (736, 3720), (913, 3552), (1056, 347), (1085, 3391), (1121, 6623), (1150, 1906), (1196, 5663), (1694, 3244), (1761, 2172), (1761, 7460), (1882, 6151), (1893, 626), (2014, 433), (2065, 3157), (2132, 378), (2161, 2352), (2200, 7485), (2393, 5191), (2489, 2519), (2508, 1575), (2509, 7089), (2599, 3151), (2645, 2672), (2782, 3380), (2906, 3676), (3072, 2934), (3133, 3418), (3188, 1653), (3624, 7812), (3661, 3603), (3694, 2929), (3759, 3418), (4155, 630), (4233, 4753), (4423, 1377), (4427, 6635), (4462, 7392), (4715, 6908), (4856, 2559), (4898, 3371), (5051, 2268), (5064, 5969), (5071, 2019), (5107, 3533), (5172, 5478), (5294, 4210), (5305, 1512), (5310, 2846), (5365, 3715), (5447, 6215), (5513, 5017), (5549, 297), (5579, 4076), (5623, 5044), (5688, 3614), (5728, 1802), (5747, 7801), (5758, 4377), (5779, 4148), (5784, 3239), (5812, 5091), (5862, 4664), (5897, 4963), (6299, 4702), (6320, 6936), (6462, 2844), (6615, 4979), (6726, 5970), (6754, 7652), (6765, 714), (6826, 3162), (6827, 3770), (6844, 1170), (6884, 226), (7023, 213), (7094, 6472), (7157, 647), (7196, 7710), (7293, 7588), (7495, 5912), (7693, 3966), (7718, 7759), (7737, 6002), (7745, 1347), (7889, 2850)]
    nCols = 8000
    nRows = 8000
    ratzarr = None
