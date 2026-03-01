# rios.imagereader
    Contains the functions needed for opening and reading input files, and the
    ReadWorkerMgr class used to manage concurrent read workers.
    
    Also contains the now-deprecated ImageReader class.

## Classes
### class ImageIterator
      Class to allow iteration across an ImageReader instance.
      Do not instantiate this class directly - it is created
      by ImageReader.__iter__().
      
      See http://docs.python.org/library/stdtypes.html#typeiter
      for a description of how this works. There is another way,
      see: http://docs.python.org/reference/expressions.html#yieldexpr
      but it seemed too much like Windows 3.1 programming which
      scared me!
      
      Returns a tuple containing an ReaderInfo class, plus a numpy
      array for each iteration

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageIterator.\_\_init\_\_(self, reader)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageIterator.next(self)

### class ImageReader
      Class that reads a single file, a list or dictionary of files and 
      iterates through them block by block
      
      **Example**
      
      ::
      
          import sys
          from rios.imagereader import ImageReader
      
          reader = ImageReader(sys.argv[1]) 
          for (info, block) in reader:     
              block2 = block * 2

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageReader.\_\_init\_\_(self, imageContainer, footprint=0, windowxsize=256, windowysize=256, overlap=0, loggingstream=<_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>, layerselection=None)
          imageContainer is a filename or list or dictionary that contains
          the filenames of the images to be read.
          If a list is passed, a list of blocks is returned at 
          each iteration, if a dictionary a dictionary is
          returned at each iteration with the same keys.
          
          footprint can be either INTERSECTION, UNION or BOUNDS_FROM_REFERENCE
          
          windowxsize and windowysize specify the size
          of the block to be read at each iteration
          
          overlap specifies the number of pixels to overlap
          between each block
          
          Set loggingstream to a file like object if you wish
          logging about resampling to be sent somewhere else
          rather than stdout.
          
          layerselection, if given, should be of the same type as imageContainer, 
          that is, if imageContainer is a dictionary, then layerselection 
          should be a dictionary with the same keys, and if imageContainer 
          is a list, then layerselection should be a list of the same length. 
          The elements in layerselection should always be lists of layer numbers, 
          used to select only particular layers to read from the corresponding 
          input image. Layer numbers use GDAL conventions, i.e. start at 1. 
          Default reads all layers. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageReader.allowResample(self, resamplemethod='near', refpath=None, refgeotrans=None, refproj=None, refNCols=None, refNRows=None, refPixgrid=None, tempdir='.', useVRT=False, allowOverviewsGdalwarp=False)
          By default, resampling is disabled (all datasets must
          match). Calling this enables it. 
          Either refgeotrans, refproj, refNCols and refNRows must be passed, 
          or refpath passed and the info read from that file.
          
          tempdir is the temporary directory where the resampling happens. By 
          default the current directory.
          
          resamplemethod is the method used - must be supported by gdalwarp.
          This can be a single string if all files are to be resampled by the
          same method, or a list or dictionary (to match what passed to the 
          constructor) contain the methods for each file.
          
          If resampling is needed it will happen before the call returns.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageReader.close(self)
          Closes all open datasets

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageReader.prepare(self, workingGrid=None)
          Prepare to read from images. These steps are not
          done in the constructor, but are done just before
          reading in case allowResample() is called which
          will resample the inputs.
          
          The pixelGrid instance to use as the working grid can
          be passed in case it is not to be derived from the images
          to be read or is different from that passed to allowResample

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageReader.readBlock(self, nblock)
          Read a block. This is normally called from the
          __getitem__ method when this class is indexed, 
          or from the ImageIterator when this class is 
          being iterated through.
          
          A block is read from each image and returned
          in a tuple along with a ReaderInfo instance.
          
          nblock is a single index, and will be converted
          to row/column.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ImageReader.readBlockWithMargin(ds, xoff, yoff, xsize, ysize, datatype, margin=0, nullValList=None, layerselection=None)
          A 'drop-in' look-alike for the ReadAsArray function in GDAL,
          but with the option of specifying a margin width, such that
          the block actually read and returned will be larger by that many pixels. 
          The returned array will ALWAYS contain these extra rows/cols, and 
          if they do not exist in the file (e.g. because the margin would push off 
          the edge of the file) then they will be filled with the given nullVal. 
          Otherwise they will be read from the file along with the rest of the block. 
          
          Variables within this function which have _margin as suffix are intended to 
          designate variables which include the margin, as opposed to those without. 
          
          This routine will cope with any specified region, even if it is entirely outside
          the given raster. The returned block would, in that case, be filled
          entirely with the null value. 

### class ReadWorkerMgr
      Simple class to hold all the things we need to sustain for
      the read worker threads

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ReadWorkerMgr.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ReadWorkerMgr.readWorkerFunc(readTaskQue, blockBuffer, controls, tmpfileMgr, rasterizeMgr, workinggrid, allInfo, timings, forceExit, exceptionQue)
          This function runs in each read worker thread. The readTaskQue gives
          it tasks to perform (i.e. single blocks of data to read), and it loops
          until there are no more to do. Each block is sent back through
          the blockBuffer.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ReadWorkerMgr.shutdown(self)
          Shut down the read worker manager

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ReadWorkerMgr.startReadWorkers(self, blockList, infiles, allInfo, controls, tmpfileMgr, rasterizeMgr, workinggrid, inBlockBuffer, timings, exceptionQue)
          Start the requested number of read worker threads, within the current
          process. All threads will read single blocks from individual files
          and place them into the inBlockBuffer.
          
          Return value is an instance of ReadWorkerMgr, which must remain
          active until all reading is complete.

### class str
      str(object='') -> str
      str(bytes_or_buffer[, encoding[, errors]]) -> str
      
      Create a new string object from the given object. If encoding or
      errors is specified, then the object must expose a data buffer
      that will be decoded using the given encoding and error handler.
      Otherwise, returns the result of object.__str__() (if defined)
      or repr(object).
      encoding defaults to sys.getdefaultencoding().
      errors defaults to 'strict'.

## Functions
### def openForWorkingGrid(filename, workinggrid, fileInfo, controls, tmpfileMgr, rasterizeMgr, symbolicName)
        If the fileInfo for the given filename is a raster, aligned with
        the working grid, just open it. If it is a raster, but not aligned,
        do a warp VRT that makes it aligned, and open that instead.
        If it is a vector, then first rasterize into a temp file and use that.
        
        Either way, return a GDAL Dataset object and a list of band objects
        corresponding to the selected bands.

### def readBlockAllFiles(infiles, workinggrid, blockDefn, allInfo, gdalObjCache, controls, tmpfileMgr, rasterizeMgr)
        Read all input files for a single block.
        Return the complete BlockAssociations object (i.e. 'inputs').

### def readBlockOneFile(blockDefn, symbolicName, seqNum, filename, gdalObjCache, controls, tmpfileMgr, rasterizeMgr, workinggrid, allInfo)
        Read the requested block, as per blockDefn, of the requested file,
        as per (symbolicName, seqNum, filename). If the file has already been
        opened, its GDAL objects will be in the gdalObjCache, otherwise
        it will be opened and those objects placed in the cache.
        
        Return a numpy array for the block, of shape (numBands, numRows, numCols).

### def readIntoArray(outArray, ds, bandObj, top_wg, left_wg, xsize, ysize, workinggrid, margin)
        Read the requested block from the given band/dataset, and place it
        into the given output array. If the block falls off the edge of the
        file extent, the request is trimmed back, and the resulting smaller
        block is placed into the correct part of the array, leaving the
        surrounding array elements unchanged.
        
        The request coordinates (top, left, xsize, ysize) do not include the
        margin (i.e. overlap), so that is accounted for explicitly here.
        If margin > 0, the array is thus larger by (2*margin) in each direction.
        
        NOTE: While it may seem that this could be done using a VRT, our tests
        of that approach found that it imposes a substantial overhead, and doing
        it ourselves is much faster. 

### def reprojResolution(xRes, yRes, x, y, srcSRS, tgtSRS)
        Return a reprojected version of the given resolution. The (xRes yRes)
        values are given in the srcSRS project, and are translated to something
        as similar as possible in the tgtSRS projection. The rough location
        is given by (x, y) (in the src projection), so the transformation is
        at its best around that point, and would be progressively worse the
        further one gets from there (due to the increased distortion from the
        different projections).

### def reprojectionRequired(imgInfo, workinggrid)
        Compare the details of the given imgInfo and the workinggrid,
        to work out if a reprojection is required. Return True if so.

### def specialProjFixes(projwkt)
        Does any special fixes required for the projection. Returns the fixed
        projection WKT string.
        
        Specifically this does two things, both of which are to cope with rubbish
        that Imagine has put into the projection. Firstly, it removes the
        crappy TOWGS84 parameters which Imagine uses for GDA94, and secondly
        removes the crappy name which Imagine gives to the correct GDA94.
        
        If neither of these things is found, returns the string unchanged.

## Values
    DEFAULTFOOTPRINT = 0
    DEFAULTLOGGINGSTREAM = <_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
    DEFAULTOVERLAP = 0
    DEFAULTWINDOWXSIZE = 256
    DEFAULTWINDOWYSIZE = 256
