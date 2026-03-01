# pyshepseg.tiling
    Routines in support of tiled segmentation of very large rasters. 
    
    Main entry routine is doTiledShepherdSegmentation(). See that
    function for further details. 
    
    The broad idea is that the Shepherd segmentation algorithm, as
    implemented in the shepseg module, runs entirely in memory. 
    For larger raster files, it is more efficient to divide the raster 
    into tiles, segment each tile individually, and stitch the
    results together to create a segmentation of the whole raster. 
    
    The main caveats arise from the fact that the initial clustering
    is performed on a uniform subsample of the whole image, in 
    order to give consistent segment boundaries at tile intersections. 
    This means that for a larger raster, with a greater range of 
    spectra, one may wish to increase the number of clusters in order 
    to allow sufficient initial segments to characterize the variation. 
    
    Related to this, one may also consider reducing the percentile
    used for automatic estimation of maxSpectralDiff (see 
    :func:`pyshepseg.shepseg.doShepherdSegmentation` and
    :func:`pyshepseg.shepseg.autoMaxSpectralDiff` for further details).
    
    Because of these caveats, one should be very cautious about 
    segmenting something like a continental-scale image. There is a 
    lot of spectral variation across an area like a whole continent, 
    and it may be unwise to use all the same parameters for the
    whole area. 

## Classes
### class FargateConfig
      Configuration for AWS Fargate (i.e. for use with CONC_FARGATE).
      
      Parameters
      ----------
        containerImage : str
          URI of the container image to use for segmentation workers. This
          container must have pyshepseg installed. It can be the same
          container as used for the main script, as the entry point is
          over-written.
        taskRoleArn : str
          ARN for an AWS role. This allows your code to use AWS services.
          This role should include policies such as AmazonS3FullAccess,
          covering any AWS services the segmentation workers will need.
        executionRoleArn : str
          ARN for an AWS role. This allows ECS to use AWS services on
          your behalf. A good start is a role including
          AmazonECSTaskExecutionRolePolicy
        subnet : str
          Subnet ID string associated with the VPC in which workers will
          run.
        securityGroups : list of str
          Fargate. List of security group IDs associated with the VPC.
        cpu : str
          Number of CPU units requested for each segmentation worker,
          expressed in AWS's own units. For example, '0.5 vCPU', or
          '1024' (which corresponds to the same thing). Both must be strings.
          This helps Fargate to select a suitable VM instance type.
        memory : str
          Amount of memory requested for each segmentation worker,
          expressed in MiB, or with a units suffix. For example, '1024'
          or its equivalent '1GB'. This helps Fargate to select a suitable
          VM instance type.
        cpuArchitecture : str
          If given, selects the CPU architecture of the hosts to run
          worker on. Can be 'ARM64', defaults to 'X86_64'.
        cloudwatchLogGroup : str or None
          If not None, the name of a CloudWatch log group. This group should
          already exist, in the region that the job is running. Logs from
          workers will be sent to this log group. If None, no CloudWatch
          logging is done. Intended for tracking problems, rather than
          operational use.
        tags : dict or None
          Optional. If specified this needs to be a dictionary of key/value
          pairs which will be turned into AWS tags. These will be added to
          the ECS cluster, task definition and tasks. The keys and values
          must all be strings. Requires ``ecs:TagResource`` permission.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FargateConfig.\_\_init\_\_(self, containerImage=None, taskRoleArn=None, executionRoleArn=None, subnet=None, securityGroups=None, cpu='0.5 vCPU', memory='1GB', cpuArchitecture=None, cloudwatchLogGroup=None, tags=None)
          AWS Fargate configuration information. For use only with CONC_FARGATE.

### class HistogramAccumulator
      Accumulator for histogram for the output segmentation image. This
      allows us to accumulate the histogram incrementally, tile-by-tile.
      Note that there are simplifying assumptions about being uint32, and
      the null value being zero, so don't try to use this for anything else.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HistogramAccumulator.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HistogramAccumulator.addTwoHistograms(hist1, hist2)
          Add the two given histograms together, and return the result.
          
          If one is longer than the other, the shorter one is added to it.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HistogramAccumulator.doHistAccum(self, arr)
          Accumulate the histogram with counts from the given arr.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HistogramAccumulator.updateHist(self, newCounts)
          Update the current histogram counts. If positive is True, then
          the counts for positive values are updated, otherwise those for the
          negative values are updated.

### class NetworkDataChannel
      Single class to manage communication with workers running on different
      machines. Uses the facilities in multiprocessing.managers.
      
      Created from either the server or the client end, the constructor
      takes 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NetworkDataChannel.\_\_init\_\_(self, inQue=None, segResultCache=None, forceExit=None, exceptionQue=None, segDataDict=None, readSemaphore=None, timings=None, workerBarrier=None, hostname=None, portnum=None, authkey=None)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NetworkDataChannel.addressStr(self)
          Return a single string encoding the network address of this channel

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NetworkDataChannel.shutdown(self)
          Shut down the NetworkDataChannel in the right order. This should always
          be called explicitly by the creator, when it is no longer
          needed. If left to the garbage collector and/or the interpreter
          exit code, things are shut down in the wrong order, and the
          interpreter hangs on exit.
          
          I have tried __del__, also weakref.finalize and atexit.register,
          and none of them avoid these problems. So, just make sure you
          call shutdown explicitly, in the process which created the
          NetworkDataChannel.
          
          The client processes don't seem to care, presumably because they
          are not running the server thread. Calling shutdown on the client
          does nothing.

### class PyShepSegTilingError(Exception)
      Common base class for all non-exit exceptions.

### class SegFargateMgr(SegmentationConcurrencyMgr)
      Run tiled segmentation with concurrency based on AWS Fargate workers.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.\_\_init\_\_(self, infile, outfile, tileSize, overlapSize, minSegmentSize, numClusters, bandNumbers, subsamplePcnt, maxSpectralDiff, imgNullVal, fixedKMeansInit, fourConnected, verbose, simpleTileRecode, outputDriver, creationOptions, spectDistPcntile, kmeansObj, tempfilesDriver, tempfilesCreationOptions, writeHistogram, returnGDALDS, concCfg)
          Constructor. Just saves all its arguments to self, and does a couple
          of quick checks.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.checkTaskErrors(self)
          Check for errors reported via describe_tasks(). This mechanism
          seems rather unreliable, particularly when reporting the 'reason',
          but I am doing my best.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.getClusterTaskCount(self)
          Query the cluster, and return the number of tasks it has.
          This is the total of running and pending tasks.
          If the cluster does not exist, return None.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.shutdown(self)
          Shut down the workers and data channel

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.specificChecks(self)
          Initial checks which are specific to the subclass

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.startWorkers(self)
          Start all segmentation workers as AWS Fargate tasks

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegFargateMgr.waitClusterTasksFinished(self)
          Poll the given cluster until the number of tasks reaches zero

### class SegNoConcurrencyMgr(SegmentationConcurrencyMgr)
      Runs tiled segmentation with no concurrency

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.\_\_init\_\_(self, infile, outfile, tileSize, overlapSize, minSegmentSize, numClusters, bandNumbers, subsamplePcnt, maxSpectralDiff, imgNullVal, fixedKMeansInit, fourConnected, verbose, simpleTileRecode, outputDriver, creationOptions, spectDistPcntile, kmeansObj, tempfilesDriver, tempfilesCreationOptions, writeHistogram, returnGDALDS, concCfg)
          Constructor. Just saves all its arguments to self, and does a couple
          of quick checks.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.checkWorkerExceptions(self)
          Dummy. No workers, so no worker exceptions.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.getTileSegmentation(self, col, row)
          Read the requested tile of segmentation output from disk

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.loadOverlap(self, overlapCacheKey)
          Load the requested overlap from disk cache

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.overlapCacheFilename(self, overlapCacheKey)
          Return filename for given overlapCacheKey

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.saveOverlap(self, overlapCacheKey, overlapData)
          Save given overlap data to disk file

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.segmentAllTiles(self)
          Run segmentation for all tiles, and write output image. Just runs all
          tiles in sequence, and then the recode and stitch together for final
          output.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegNoConcurrencyMgr.writeTileToTemp(self, segResult, filename, outDrvr, xpos, ypos, xsize, ysize)
          Write the segmented tile to a temporary image file

### class SegSubprocMgr(SegmentationConcurrencyMgr)
      Run tiled segmentation with concurrency based on subprocess workers.
      This is used only as a test bed for the NetworkDataChannel and external
      worker command, and should not be used in real life.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegSubprocMgr.\_\_init\_\_(self, infile, outfile, tileSize, overlapSize, minSegmentSize, numClusters, bandNumbers, subsamplePcnt, maxSpectralDiff, imgNullVal, fixedKMeansInit, fourConnected, verbose, simpleTileRecode, outputDriver, creationOptions, spectDistPcntile, kmeansObj, tempfilesDriver, tempfilesCreationOptions, writeHistogram, returnGDALDS, concCfg)
          Constructor. Just saves all its arguments to self, and does a couple
          of quick checks.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegSubprocMgr.startWorkers(self)
          Start all segmentation workers

### class SegThreadsMgr(SegmentationConcurrencyMgr)
      Run tiled segmentation with concurrency based on threads within the main
      process.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegThreadsMgr.\_\_init\_\_(self, infile, outfile, tileSize, overlapSize, minSegmentSize, numClusters, bandNumbers, subsamplePcnt, maxSpectralDiff, imgNullVal, fixedKMeansInit, fourConnected, verbose, simpleTileRecode, outputDriver, creationOptions, spectDistPcntile, kmeansObj, tempfilesDriver, tempfilesCreationOptions, writeHistogram, returnGDALDS, concCfg)
          Constructor. Just saves all its arguments to self, and does a couple
          of quick checks.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegThreadsMgr.setupNetworkComms(self)
          Dummy. No network communications required.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegThreadsMgr.shutdown(self)
          Shut down the thread pool

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegThreadsMgr.specificChecks(self)
          Checks which are specific to the subclass

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegThreadsMgr.startWorkers(self)
          Start worker threads for segmenting tiles

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegThreadsMgr.worker(self)
          Worker function. Called for each worker thread.

### class SegmentationConcurrencyConfig
      Configuration for concurrency in segmentation of multiple tiles.
      
      The segmentation of each tile can be performed concurrently by individual
      workers. However, the stitching together of the resulting tiles is
      inherently sequential, and with sufficient workers, this easily becomes
      the dominant operation. Adding more workers after this simply increases
      the memory usage for tiles waiting to be stitched (up to segResultCacheSize),
      without any further speedup.
      
      It is recommended that the user begin with a small number of workers, and
      inspect the timings (see :func:`pyshepseg.utils.formatTimingRpt`) and
      increase the number of workers so as to reduce ``stitchwaitfortile`` time.
      When this no longer decreases, there is no further benefit to adding more
      workers.
      
      Parameters
      ----------
        concurrencyType : One of {CONC_NONE, CONC_THREADS, CONC_FARGATE, CONC_SUBPROC}
          The mechanism used for concurrency
        numWorkers : int
          Number of segmentation workers
        maxConcurrentReads : int
          Maximum number of concurrent reads. Each segmentation worker
          does its own reading of input data. Since the number of workers
          can be quite large, this could load the read device too heavily.
          Given that the read step is a very small component of each
          worker's activity, we can limit the number of concurrent reads
          to this value, without degrading throughput.
        tileCompletionTimeout : int
          Timeout (seconds) to wait for completion of each segmentation tile
        segResultCacheSize : int
          Maximum number of completed tile segmentations in cache
        segResultCacheAddTimeout : int
          Timeout (seconds) to wait to add a completed segmentation into
          the result cache. If this timeout is reached, it may indicate that there
          are too many workers.
        barrierTimeout : int
          Timeout (seconds) to wait for all workers to start. Used with
          CONC_FARGATE (and CONC_SUBPROC).
        fargateCfg : None or instance of FargateConfig
          Configuration for AWS Fargate (when using CONC_FARGATE)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyConfig.\_\_init\_\_(self, concurrencyType='CONC_NONE', numWorkers=0, maxConcurrentReads=20, tileCompletionTimeout=60, segResultCacheSize=30, segResultCacheAddTimeout=300, barrierTimeout=300, fargateCfg=None)
          Configuration for managing segmentation concurrency.

### class SegmentationConcurrencyMgr
      Base class for segmentation concurrency

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.\_\_init\_\_(self, infile, outfile, tileSize, overlapSize, minSegmentSize, numClusters, bandNumbers, subsamplePcnt, maxSpectralDiff, imgNullVal, fixedKMeansInit, fourConnected, verbose, simpleTileRecode, outputDriver, creationOptions, spectDistPcntile, kmeansObj, tempfilesDriver, tempfilesCreationOptions, writeHistogram, returnGDALDS, concCfg)
          Constructor. Just saves all its arguments to self, and does a couple
          of quick checks.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.checkForEmptySegments(self, hist, overlapSize)
          Check the final segmentation for any empty segments. These
          can be problematic later, and should be avoided. Prints a
          warning message if empty segments are found.
          
          Parameters
          ----------
            hist : ndarray of uint32
              Histogram counts for the segmentation raster
            overlapSize : int
              Number of pixels to use in overlaps between tiles
          
          Returns
          -------
            hasEmptySegments : bool
              True if there are segment ID numbers with no pixels

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.checkWorkerExceptions(self)
          Check if any workers raised exceptions. If so, raise a local exception
          with the WorkerErrorRecord.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.crossesMidline(overlap, segLoc, orientation)
          Check whether the given segment crosses the midline of the
          given overlap. If it does not, then it will lie entirely within
          exactly one tile, but if it does cross, then it will need to be
          re-coded across the midline.
          
          Parameters
          ----------
            overlap : shepseg.SegIdType ndarray (overlapNrows, overlapNcols)
              Array of segments just for this overlap region
            segLoc : shepseg.RowColArray
              The row/col coordinates (within the overlap array) of the
              pixels for the segment of interest
            orientation : {HORIZONTAL, VERTICAL}
              Indicates the orientation of the midline
          
          Returns
          -------
            crosses : bool
              True if the given segment crosses the midline

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.getTileSegmentation(self, col, row)
          Get the segmented tile output data from the local cache, and remove it
          from the cache

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.initialize(self)
          Runs initial phase of segmentation. This does not have any concurrency,
          so is the same for every concurrencyType. The main job is to do the
          spectral clustering, setting self.kmeansObj

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.loadOverlap(self, overlapCacheKey)
          Load the requested overlap from cache, and remove it from cache

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.overlapCacheKey(col, row, edge)
          Return the temporary cache key used for the overlap array
          
          Parameters
          ----------
            col, row : int
              Tile column & row numbers
            edge : {right', 'bottom'}
              Indicates from which edge of the given tile the overlap is taken
          
          Returns
          -------
            cachekey : str
              Identifying key for the overlap

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.popFromQue(que)
          Pop out the next item from the given Queue, returning None if
          the queue is empty.
          
          WARNING: don't use this if the queued items can be None

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.recodeSharedSegments(tileData, overlapA, overlapB, orientation, recodeDict)
          Work out a mapping which recodes segment ID numbers from
          the tile in tileData. Segments to be recoded are those which 
          are in the overlap with an earlier tile, and which cross the 
          midline of the overlap, which is where the stitchline between 
          the tiles will fall. 
          
          Updates recodeDict, which is a dictionary keyed on the 
          existing segment ID numbers, where the value of each entry 
          is the segment ID number from the earlier tile, to be used 
          to recode the segment in the current tile. 
          
          overlapA and overlapB are numpy arrays of pixels in the overlap
          region in question, giving the segment ID numbers in the two tiles.
          The values in overlapB are from the earlier tile, and those in
          overlapA are from the current tile.
          
          It is critically important that the overlapping region is either
          at the top or the left of the current tile, as this means that 
          the row and column numbers of pixels in the overlap arrays 
          match the same pixels in the full tile. This cannot be used
          for overlaps on the right or bottom of the current tile.
          
          Parameters
          ----------
            tileData : shepseg.SegIdType ndarray (tileNrows, tileNcols)
              Tile subset of segment ID image
            overlapA, overlapB : shepseg.SegIdType ndarray (overlapNrows, overlapNcols)
              Tile overlap subsets of segment ID image
            orientation : {HORIZONTAL, VERTICAL}
              The orientation parameter defines whether we are dealing with
              overlap at the top (orientation == HORIZONTAL) or the left
              (orientation == VERTICAL).
            recodeDict : dict
              Keys and values are both segment ID numbers. Defines the mapping
              which recodes segment IDs. Updated in place.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.recodeTile(self, tileData, maxSegId, tileRow, tileCol, top, bottom, left, right)
          Adjust the segment ID numbers in the current tile, to make them
          globally unique (and contiguous) across the whole mosaic.
          
          Make use of the overlapping regions of tiles above and left,
          to identify shared segments, and recode those to segment IDs 
          from the adjacent tiles (i.e. we change the current tile, not 
          the adjacent ones). Non-shared segments are increased so they 
          are larger than previous values. 
          
          Parameters
          ----------
            tileData : shepseg.SegIdType ndarray (tileNrows, tileNcols)
              The array of segment IDs for a single image tile
            maxSegId : shepseg.SegIdType
              The current maximum segment ID for all preceding tiles.
            tileRow, tileCol : int
              The row/col numbers of this tile, within the whole-mosaic
              tile numbering scheme. (These are not pixel numbers, but tile
              grid numbers)
            top, bottom, left, right : int
              Pixel coordinates *within tile* of the non-overlap region of
              the tile.
          
          Returns
          -------
            newTileData : shepseg.SegIdType ndarray (tileNrows, tileNcols)
              A copy of tileData, with new segment ID numbers.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.relabelSegments(tileData, recodeDict, maxSegId, top, bottom, left, right)
          Recode the segment IDs in the given tileData array.
          
          For segment IDs which are keys in recodeDict, these
          are replaced with the corresponding entry. For all other 
          segment IDs, they are replaced with sequentially increasing
          ID numbers, starting from one more than the previous
          maximum segment ID (maxSegId). 
          
          A re-coded copy of tileData is created, the original is 
          unchanged.
          
          Parameters
          ----------
            tileData : shepseg.SegIdType ndarray (tileNrows, tileNcols)
              Segment IDs of tile
            recodeDict : dict
              Keys and values are segment ID numbers. Defines mapping
              for segment relabelling
            maxSegId : shepseg.SegIdType
              Maximum segment ID number
            top, bottom, left, right : int
              Pixel coordinates *within tile* of the non-overlap region of
              the tile.
          
          Returns
          -------
              newTileData : shepseg.SegIdType ndarray (tileNrows, tileNcols)
                Segment IDs of tile, after relabelling
              newMaxSegId : shepseg.SegIdType
                New maximum segment ID after relabelling

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.saveOverlap(self, overlapCacheKey, overlapData)
          Save given overlap data to cache

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.segmentAllTiles(self)
          Run segmentation for all tiles, and write output image. Runs a number
          of segmentation workers, each working independently on individual
          tiles. The tiles to process are sent via a Queue, and the computed
          results are returned via the SegmentationResultCache.
          
          Stitching the tiles together is run in the main thread, beginning as
          soon as the first tile is completed.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.setupNetworkComms(self)
          Set up a NetworkDataChannel to communicate with the workers
          outside the main process (e.g. Fargate instances)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.setupOverviews(self, outDs)
          Calculate a suitable set of overview levels to use for output
          segmentation file, and set these up on the given Dataset. Stores
          the overview levels list as self.overviewLevels

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.shutdown(self)
          Any explicit shutdown operations

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.specificChecks(self)
          Checks which are specific to the subclass. Called at the
          end of __init__().

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.startWorkers(self)
          Start segmentation workers, if required

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.stitchTiles(self)
          Recombine individual tiles into a single segment raster output 
          file. Segment ID values are recoded to be unique across the whole
          raster, and contiguous.
          
          Sets maxSegId and outDs on self.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.writeHistogramToFile(outBand, histAccum)
          Write the accumulated histogram to the output segmentation file

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationConcurrencyMgr.writeOverviews(self, outBand, arr, xOff, yOff)
          Calculate and write out the overview layers for the tile
          given as arr.

### class SegmentationResultCache
      Thread-safe cache for segmentation results, by tile. As each worker completes
      a tile, it adds it directly to this cache. The writing thread can then
      pop tiles out of this when required.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationResultCache.\_\_init\_\_(self, colRowList, timeout=None, size=10, addTimeout=300)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationResultCache.addResult(self, col, row, segResult)
          Add a single segResult object to the cache, for the given (col, row)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SegmentationResultCache.waitForTile(self, col, row)
          Wait until the nominated tile is ready, and then pop it out of
          the cache.

### class TileInfo
      Class that holds the pixel coordinates of the tiles within 
      an image. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TileInfo.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TileInfo.addTile(self, xpos, ypos, xsize, ysize, col, row)
          Add a new tile to the set
          
          Parameters
          ----------
            xpos, ypos : int
              Pixel column & row of top left pixel of tile
            xsize, ysize : int
              Number of pixel columns & rows in tile
            col, row : int
              Tile column & row

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TileInfo.getNumTiles(self)
          Get total number of tiles in the set
          
          Returns
          -------
            numTiles : int
              Total number of tiles

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TileInfo.getTile(self, col, row)
          Return the position and shape of the requested tile, as
          a single tuple of values
          
          Parameters
          ----------
            col, row : int
              Tile column & row
          
          Returns
          -------
            xpos, ypos : int
              Pixel column & row of top left pixel of tile
            xsize, ysize : int
              Number of pixel columns & rows in tile

### class TiledSegmentationResult
      Result of tiled segmentation
      
      Attributes
      ----------
        maxSegId : shepseg.SegIdType
          Largest segment ID used in final segment image
        numTileRows : int
          Number of rows of tiles used
        numTileCols : int
          Number of columns of tiles used
        subsamplePcnt : float
          Percentage of image subsampled for clustering
        maxSpectralDiff : float
          The value used to limit segment merging (in all tiles)
        kmeans : sklearn.cluster.KMeans
          The sklearn KMeans object, after fitting
        hasEmptySegments : bool
          True if the segmentation contains segments with no pixels.
          This is an error condition, probably indicating that the
          merging of segments across tiles has produced inconsistent
          numbering. A warning message will also have been printed.
        timings : pyshepseg.timinghooks.Timers
          Timings for various key parts of the segmentation process
        outDs : gdal.Dataset or None
          Open GDAL dataset object to the output file. May be None,
          see the returnGDALDS parameter to doTiledShepherdSegmentation.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TiledSegmentationResult.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def calcHistogramTiled(segfile, maxSegId, writeToRat=True)
        This function is now deprecated, and will probably be removed in
        a future version.
        
        Calculate a histogram of the given segment image file.
        
        Note that we need this function because GDAL's GetHistogram
        function does not seem to work when attempting a histogram
        with very large numbers of entries. We want an entry for
        every segment, rather than an approximate count for a range of
        segment values, and the number of segments is very large. So,
        we need to write our own routine. 
        
        It works in tiles across the image, so that it can process
        very large images in a memory-efficient way.
        
        For a raster which can easily fit into memory, a histogram
        can be calculated directly using :func:`pyshepseg.shepseg.makeSegSize`.
        
        Parameters
        ----------
          segfile : str or gdal.Dataset
            Segmentation image file. Can be either the file name string, or
            an open Dataset object.
          maxSegId : shepseg.SegIdType
            Maximum segment ID used
          writeToRat : bool
            If True, the completed histogram will be written to the image
            file's raster attribute table. If segfile was given as a Dataset
            object, it would therefore need to have been opened with update
            access.
        
        Returns
        -------
          hist : int ndarray (numSegments+1, )
            Histogram counts for each segment (index is segment ID number)

### def doTiledShepherdSegmentation(infile, outfile, tileSize=4096, overlapSize=1024, minSegmentSize=50, numClusters=60, bandNumbers=None, subsamplePcnt=None, maxSpectralDiff='auto', imgNullVal=None, fixedKMeansInit=False, fourConnected=True, verbose=False, simpleTileRecode=False, outputDriver='KEA', creationOptions=[], spectDistPcntile=50, kmeansObj=None, tempfilesDriver='KEA', tempfilesExt='kea', tempfilesCreationOptions=[], writeHistogram=True, returnGDALDS=False, concurrencyCfg=None)
        Run the Shepherd segmentation algorithm in a memory-efficient
        manner, suitable for large raster files. Runs the segmentation
        on separate (overlapping) tiles across the raster, then stitches these
        together into a single output segment raster.
        
        The initial spectral clustering is performed on a sub-sample
        of the whole raster (using fitSpectralClustersWholeFile), 
        to create consistent clusters. These are then used as seeds 
        for all individual tiles. Note that subsamplePcnt is used at 
        this stage, over the whole raster, and is not passed through to 
        shepseg.doShepherdSegmentation() for any further sub-sampling.
        
        Most of the arguments are passed through to 
        shepseg.doShepherdSegmentation, and are described in the docstring
        for that function.
        
        Parameters
        ----------
          infile : str
            Filename of input raster
          outfile : str
            Filename of output segmentation raster
          tileSize : int
            Desired width & height (in pixels) of the tiles (i.e.
            desired tiles have shape (tileSize, tileSize). Tiles on the
            right and bottom edges of the input image may end up slightly
            larger than tileSize to ensure there are no small tiles.
          overlapSize : int
            Number of pixels to overlap tiles. The overlap area is a rectangle,
            this many pixels wide, which is covered by both adjacent tiles.
          minSegmentSize : int
            Minimum number of pixels in a segment
          numClusters : int
            Number of clusters to request in k-means clustering
          bandNumbers : list of int
            The GDAL band numbers (i.e. start at 1) of the bands of input raster
            to use for segmentation
          subsamplePcnt : float or None
            See fitSpectralClustersWholeFile()
          maxSpectralDiff : float or str
            See shepseg.doShepherdSegmentation()
          spectDistPcntile : int
            See shepseg.doShepherdSegmentation()
          imgNullVal : float or None
            If given, use this as the null value for the input raster. If None,
            use the value defined in the raster file
          fixedKMeansInit : bool
            If True, use a fixed set of initial cluster centres for the KMeans
            clustering. This is good to ensure exactly reproducible results
          fourConnected : bool
            If True, use 4-way connectedness, otherwise use 8-way
          verbose : bool
            If True, print informative messages during processing (to stdout)
          simpleTileRecode : bool
            If True, use only a simple tile recoding procedure. See
            stitchTiles() for more detail
          outputDriver : str
            The short name of the GDAL format driver to use for output file
          creationOptions : list of str
            The GDAL output creation options to match the outputDriver
          kmeansObj : sklearn.cluster.KMeans
            See shepseg.doShepherdSegmentation() for details
          tempfilesDriver : str
            Short name of GDAL driver to use for temporary raster files
          tempfilesExt : str
            File extension to use for temporary raster files
          tempfilesCreationOptions : list of str
            GDAL creation options to use for temporary raster files
          writeHistogram : bool
            Deprecated, and ignored. The histogram is always written.
          returnGDALDS : bool
            Whether to set the outDs member of TiledSegmentationResult
            when returning. If set, this will be open in update mode.
          concurrencyCfg : SegmentationConcurrencyConfig
            Configuration for segmentation concurrency. Default is None,
            meaning no concurrency.
        
        Returns
        -------
          tileSegResult : TiledSegmentationResult

### def fitSpectralClustersWholeFile(inDs, bandNumbers, numClusters=60, subsamplePcnt=None, imgNullVal=None, fixedKMeansInit=False)
        Given an open raster Dataset, read a selected sample of pixels
        and use these to fit a spectral cluster model. Uses GDAL
        to read the pixels, and shepseg.fitSpectralClusters() 
        to do the fitting.
        
        Parameters
        ----------
          inDs : gdal.Dataset
            Open GDAL Dataset object for the input raster
          bandNumbers : list of int (or None)
            List of GDAL band numbers for the bands of interest. If
            None, then use all bands in the dataset. Note that GDAL band
            numbers start at 1.
          numClusters : int
            Desired number of clusters
          subsamplePcnt : float or None
            Percentage of pixels to use in fitting. If it is None, then
            a suitable subsample is calculated such that around one million
            pixels are sampled. (Note - this would include null pixels, so
            if the image is dominated by nulls, this would undersample.)
            No further subsampling is carried out by fitSpectralClusters().
          imgNullVal : float or None
            Pixels with this value in the input raster are ignored. If None,
            the NoDataValue from the raster file is used
          fixedKMeansInit : bool
            If True, then use a fixed estimate for the initial KMeans cluster
            centres. See shepseg.fitSpectralClusters() for details.
        
        Returns
        -------
          kmeansObj : sklearn.cluster.KMeans
            The fitted KMeans object
          subsamplePcnt : float
            The subsample percentage actually used
          imgNullVal : float
            The image null value (possibly read from the file)

### def getImgNullValue(inDs, bandNumbers)
        Return the null value for the given dataset
        
        Parameters
        ----------
          inDs : gdal.Dataset
            Open input Dataset
          bandNumbers : list of int
            GDAL band numbers of interest
        
        Returns
        -------
          imgNullVal : float or None
            Null value from input raster, None if there is no null value
        
        Raises
        ------
          PyShepSegTilingError
            If not all bands have the same null value

### def getTilesForFile(ds, tileSize, overlapSize)
        Return a TileInfo object for a given file and input
        parameters.
        
        Parameters
        ----------
          ds : gdal.Dataset
            Open GDAL Dataset object for raster to be tiles
          tileSize : int
            Size of tiles, in pixels. Individual tiles may end up being
            larger in either direction, when they meet the edge of the raster,
            to ensure we do not use very small tiles
          overlapSize : int
            Number of pixels by which tiles will overlap
        
        Returns
        -------
          tileInfo : TileInfo
            TileInfo object detailing the sizes and positions of all tiles
            across the raster

### def readSubsampledImageBand(bandObj, subsampleProp)
        Read in a sub-sampled copy of the whole of the given band. 
        
        Note that one can, in principle, do this directly using GDAL. 
        However, if overview layers are present in the file, it will use
        these, and so is dependent on how these were created. Since 
        these are often created just for display purposes, past experience
        has shown that they are not always to be trusted as data, 
        so we have chosen to always go directly to the full resolution 
        image. 
        
        Parameters
        ----------
          bandObj : gdal.Band
            An open Band object for input
          subsampleProp : float
            The proportion by which to sub-sample (i.e. a value between
            zero and 1, applied to rows and columns separately)
        
        Returns
        -------
          imgSub : <dtype> ndarray (nRowsSub, nColsSub)
            A numpy array of the image subsample, equivalent to
            calling gdal.Band.ReadAsArray()

### def selectConcurrencyClass(concurrencyType, baseClass)
        Choose the sub-class corresponding to the given concurrencyType

## Values
    BOTTOM_OVERLAP = 'bottom'
    CONC_FARGATE = 'CONC_FARGATE'
    CONC_NONE = 'CONC_NONE'
    CONC_SUBPROC = 'CONC_SUBPROC'
    CONC_THREADS = 'CONC_THREADS'
    DFLT_CHUNKSIZE = 100000
    DFLT_OVERLAPSIZE = 1024
    DFLT_TEMPFILES_DRIVER = 'KEA'
    DFLT_TEMPFILES_EXT = 'kea'
    DFLT_TILESIZE = 4096
    HORIZONTAL = 0
    RIGHT_OVERLAP = 'right'
    TILESIZE = 1024
    VERTICAL = 1
    boto3 = None
    segIdNumbaType = uint32
    updateCounts = CPUDispatcher(<function updateCounts at 0x7e6aecd32fc0>)
