# pyshepseg.shepseg
    Python implementation of the image segmentation algorithm described
    by Shepherd et al [1]_
    
    Implemented using scikit-learn's K-Means algorithm [2]_, and using
    numba [3]_ compiled code for the other main components. 
    
    Main entry point is the doShepherdSegmentation() function. 
    
    Examples
    --------
    
    Read in a multi-band image as a single array, img,
    of shape (nBands, nRows, nCols). 
    Ensure that any null pixels are all set to a known 
    null value in all bands. Failure to correctly identify 
    null pixels can result in a poorer quality segmentation. 
        
    >>> from pyshepseg import shepseg
    >>> segRes = shepseg.doShepherdSegmentation(img, imgNullVal=nullVal)
        
    The segimg attribute of the segRes object is an array
    of segment ID numbers, of shape (nRows, nCols). 
        
    Resulting segment ID numbers start from 1, and null pixels 
    are set to zero. 
    
    **Efficient segment location**
    
    After segmentation, the location of individual segments can be
    found efficiently using the object returned by makeSegmentLocations().
    
    >>> segSize = shepseg.makeSegSize(segimg)
    >>> segLoc = shepseg.makeSegmentLocations(segimg, segSize)
        
    This segLoc object is indexed by segment ID number (must be
    of type shepseg.SegIdType), and each element contains information
    about the pixels which are in that segment. This information
    can be returned as a slicing object suitable to index the image array
    
    >>> segNdx = segLoc[segId].getSegmentIndices()
    >>> vals = img[0][segNdx]
        
    This example would give an array of the pixel values from the first
    band of the original image, for the given segment ID.
    
    This can be a very efficient way to calculate per-segment
    quantities. It can be used in pure Python code, or it can be used
    inside numba jit functions, for even greater efficiency.
    
    References
    ----------
    .. [1] Shepherd, J., Bunting, P. and Dymond, J. (2019). 
       Operational Large-Scale Segmentation of Imagery Based on 
       Iterative Elimination. Remote Sensing 11(6).
       https://www.mdpi.com/2072-4292/11/6/658
    .. [2] https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html
    .. [3] https://numba.pydata.org/

## Classes
### class RowColArray
      This data structure is used to store the locations of
      every pixel in a given segment. It will be used for entries
      in a jit dictionary. This means we can quickly find all the
      pixels belonging to a particular segment.
      
      Attributes
      ----------
        idx : int
          Index of most recently added pixel
        rowcols : uint32 ndarray (length, 2)
          Row and col numbers of pixels in the segment

#### &nbsp;&nbsp;&nbsp;&nbsp; RowColArray.\__init__(self, length)
        Initialize the data structure
        
        Parameters
        ----------
          length : int
            Number of pixels in the segment

#### &nbsp;&nbsp;&nbsp;&nbsp; RowColArray.append(self, row, col)
        Add the coordinates of a new pixel in the segment
        
        Parameters
        ----------
          row : int
            Row number of pixel
          col : int
            Column number of pixel

#### &nbsp;&nbsp;&nbsp;&nbsp; RowColArray.getSegmentIndices(self)
        Return the row and column numbers of the segment pixels
        as a tuple, suitable for indexing the image array. 
        This supports selection of all pixels for a given segment. 

### class SegmentationResult
      Results of the segmentation process
      
      Attributes
      ----------
        segimg : numpy array (nRows, nCols)
          Elements are segment ID numbers (starting from 1)
        kmeans : sklearn.cluster.KMeans
          Fitted KMeans object
        maxSpectralDiff : float
          The value used to limit segment merging
        singlePixelsEliminated : int
          Number of single pixels merged to adjacent segments
        smallSegmentsEliminated : int
          Number of small segments merged into adjacent segments

#### &nbsp;&nbsp;&nbsp;&nbsp; SegmentationResult.\__init__(self)
        Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def applySpectralClusters(kmeansObj, img, imgNullVal)
        Use the given KMeans object to predict spectral clusters on 
        a whole image array. 
        
        Parameters
        ----------
          kmeansObj : sklearn.cluster.KMeans
            A fitted instance, as returned by fitSpectralClusters(). 
          img : int ndarray (nBands, nRows, nCols)
            The image to predict on
          imgNullVal : int
            Any pixels in img which have value imgNullVal will be set to
            SEGNULLVAL (i.e. zero) in the output cluster image.
        
        Returns
        -------
          segimg : int ndarray (nRows, nCols)
            The initial segment image, each element being the segment 
            ID value for that pixel

### def autoMaxSpectralDiff(km, maxSpectralDiff, distPcntile)
        Work out what to use as the maxSpectralDiff.
        
        If current value is 'auto', then return the median spectral
        distance between cluster centres from the KMeans clustering
        object km.
        
        If current value is None, return 10 times the largest distance
        between cluster centres (i.e. too large ever to make a difference)
        
        Otherwise, return the given current value.
        
        Parameters
        ----------
          km : sklearn.cluster.KMeans 
            KMeans clustering object
          maxSpectralDiff : str or float
            It is given in the units of the spectral space of img. If 
            maxSpectralDiff is 'auto', a default value will be calculated
            from the spectral distances between cluster centres, as a 
            percentile of the distribution of these (distPcntile). 
            The value of distPcntile should be lowered when segementing 
            an image with a larger range of spectral distances. 
          distPcntile : int
            See maxSpectralDiff
        
        Returns
        -------
          maxSpectralDiff : int
            The value to use as maxSpectralDiff.

### def diagonalClusterCentres(xSample, numClusters)
        Calculate an array of initial guesses at cluster centres. 
        This will be given to the KMeans constructor as the init
        parameter. 
        
        The centres are evenly spaced along the diagonal of 
        the bounding box of the data. The end points are placed 
        1 step in from the corners. 
        
        Parameters
        ----------
          xSample : int ndarray (numPoints, numBands)
            A sample of data to be used for fitting
          numClusters : int
            Number of cluster centres to be calculated
        
        Returns
        -------
          centres : int ndarray (numPoints, numBands)
            Initial cluster centres in spectral space

### def doShepherdSegmentation(img, numClusters=60, clusterSubsamplePcnt=1, minSegmentSize=50, maxSpectralDiff='auto', imgNullVal=None, fourConnected=True, verbose=False, fixedKMeansInit=False, kmeansObj=None, spectDistPcntile=50)
        Perform Shepherd segmentation in memory, on the given 
        multi-band img array.
        
        Parameters
        ----------
          img : integer ndarray of shape (nBands, nRows, nCols)
          numClusters : int
            Number of clusters to create with k-means clustering
          clusterSubsamplePcnt : int
            Passed to fitSpectralClusters(). See there for details
          minSegmentSize : int
            the minimum segment size (in pixels) which will be left
            after eliminating small segments (except for segments which
            cannot be eliminated).
          maxSpectralDiff : str or float
            sets a limit on how different segments can be and still be merged.
            It is given in the units of the spectral space of img. If 
            maxSpectralDiff is 'auto', a default value will be calculated
            from the spectral distances between cluster centres, as a 
            percentile of the distribution of these (spectDistPcntile). 
            The value of spectDistPcntile should be lowered when segementing 
            an image with a larger range of spectral distances. 
          spectDistPcntile : int
            See maxSpectralDiff
          fourConnected : bool
            If True, use 4-way connectedness when clumping, otherwise use
            8-way
          imgNullVal : int or None
            If not None, then pixels with this value in any band are set to zero
            (SEGNULLVAL) in the output segmentation. If there are null values
            in the image array, it is important to give this null value, as it can
            stringly affect the initial spectral clustering, which in turn 
            strongly affects the final segmenation.
          fixedKMeansInit : bool
            If fixedKMeansInit is True, then choose a fixed set of 
            cluster centres to initialize the KMeans algorithm. This can
            be useful to provide strict determinacy of the results by
            avoiding sklearn's multiple random initial guesses. The default 
            is to allow sklearn to guess, which is good for avoiding 
            local minima. 
          kmeansObj : sklearn.cluster.KMeans object
            By default, the spectral clustering step will be fitted using 
            the given img. However, if kmeansObj is not None, it is taken 
            to be a fitted instance of sklearn.cluster.KMeans, and will 
            be used instead. This is useful when enforcing a consistent 
            clustering across multiple tiles (see the pyshepseg.tiling 
            module for details).
        
        Returns
        -------
        segResult : SegmentationResult object
        
        Notes
        -----    
        Default values are mostly as suggested by Shepherd et al. 
        
        Segment ID numbers start from 1. Zero is a NULL segment ID. 
        
        The return value is an instance of SegmentationResult class. 
        
        See Also
        --------
        pyshepseg.tiling, fitSpectralClusters

### def eliminateSinglePixels(img, seg, segSize, minSegId, maxSegId, fourConnected)
        Approximate elimination of single pixels, as suggested 
        by Shepherd et al (section 2.3, page 6). This step suggested as 
        an efficient way of removing a large number of segments which 
        are single pixels, by approximating the spectrally-nearest
        neighbouring segment with the spectrally-nearest neighouring
        pixel. 
        
        Parameters
        ----------
          img : int ndarray (nBands, nRows, nCols)
            The original spectral image
          seg : SegIdType ndarray (nRows, nCols)
            The image of segment IDs
          segSize : int array (numSeg+1, )
            Array of pixel counts for every segment
          minSegId : SegIdType
            Smallest segment ID
          maxSegId : SegIdType
            Largest segment ID
          fourConnected : bool
            If True use 4-way connectedness, otherwise 8-way
        
        Notes
        -----
        
        Segment ID numbers start at 1 (i.e. 0 is not valid)
        
        Modifies seg array in place. 

### def fitSpectralClusters(img, numClusters, subsamplePcnt, imgNullVal, fixedKMeansInit)
        First step of Shepherd segmentation. Use K-means clustering
        to create a set of "seed" segments, labelled only with
        their spectral cluster number. 
        
        Parameters
        ----------
          img : int ndarray (nBands, nRows, nCols).
          numClusters : int
            The number of clusters for the KMeans algorithm to find
            (i.e. it is 'k') 
          subsamplePcnt : int
            The percentage of the pixels to actually use for KMeans clustering.
            Shepherd et al find that only a very small percentage is required. 
          imgNullVal : int or None
            If imgNullVal is not None, then pixels in img with this value in 
            any band are set to segNullVal in the output. 
          fixedKMeansInit : bool
            If True, then use a simple algorithm to determine the fixed
            set of initial cluster centres. Otherwise allow the sklearn 
            routine to choose its own initial guesses. 
        
        Returns
        -------
        kmeansObj : sklearn.cluster.KMeans
          A fitted object of class sklearn.cluster.KMeans. This
          is suitable to use with the applySpectralClusters() function. 

## Values
    MINSEGID = 1
    RowColArray_Type = instance.jitclass.RowColArray#743506b873d0<idx:uint32,rowcols:array(uint32, 2d, A)>
    SEGNULLVAL = 0
    buildSegmentSpectra = CPUDispatcher(<function buildSegmentSpectra at 0x743506b7d940>)
    clump = CPUDispatcher(<function clump at 0x743506b7c860>)
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    doMerge = CPUDispatcher(<function doMerge at 0x743506b94ae0>)
    eliminateSmallSegments = CPUDispatcher(<function eliminateSmallSegments at 0x743506b94540>)
    findMergeSegment = CPUDispatcher(<function findMergeSegment at 0x743506b94680>)
    findNearestNeighbourPixel = CPUDispatcher(<function findNearestNeighbourPixel at 0x743506b7d120>)
    makeSegSize = CPUDispatcher(<function makeSegSize at 0x743506b7c900>)
    makeSegmentLocations = CPUDispatcher(<function makeSegmentLocations at 0x743506b7da80>)
    mergeSinglePixels = CPUDispatcher(<function mergeSinglePixels at 0x743506b7d080>)
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
    relabelSegments = CPUDispatcher(<function relabelSegments at 0x743506b7d620>)
    spec = [('idx', uint32), ('rowcols', Array(uint32, 2, 'A', False, aligned=True))]
