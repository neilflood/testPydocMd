# rios.readerinfo
    This module contains the ReaderInfo class
    which holds information about the area being
    read and info on the current block

## Classes
### class ReaderInfo
      ReaderInfo class. Holds information about the area being
      read and info on the current block

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.\_\_init\_\_(self, workingGrid, windowxsize, windowysize, overlap, loggingstream)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getBlockCoordArrays(self)
        Return a tuple of the world coordinates for every pixel
        in the current block. Each array has the same shape as the 
        current block. Return value is a tuple::
        
            (xBlock, yBlock)
        
        where the values in xBlock are the X coordinates of the centre
        of each pixel, and similarly for yBlock. 
        
        The coordinates returned are for the pixel centres. This is 
        slightly inconsistent with usual GDAL usage, but more likely to
        be what one wants. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getBlockCount(self)
        Gets the count of the current block

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getBlockSize(self)
        Get the size of the current block. Returns a tuple::
        
            (numCols, numRows)
        
        for the current block. Mostly the same as the window size, 
        except on the edge of the raster. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getFilenameFor(self, block)
        Get the input filename of a dataset

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getGDALBandFor(self, block, band)
        Get the underlying GDAL handle for a band of a dataset
        
        This is no longer implemented, and raises an exception if called.

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getGDALDatasetFor(self, block)
        Get the underlying GDAL handle of a dataset
        
        This is no longer implemented, and raises an exception if called.

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getNoDataValueFor(self, block, band=1)
        Returns the 'no data' value for the dataset underlying the block.
        This should be the same as what was set for the stats ignore value
        when that dataset was created. The value is cast to the same data
        type as the dataset.
        
        The band number starts at 1, following GDAL's convention.
        
        Note, however, that if controls.selectInputImageLayers() was used to
        read a reduced set of input layers from the file, then these numbers
        are of the reduced set. For example, 1 will refer to the first of the
        selected layers, which may not be the first in the file.
        
        This is not the preferred method for the user function to access
        the null value for an input file. A more transparent approach is to
        make such values available on the otherArgs object. This function is
        maintained for backward compatibility.

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getOverlapSize(self)
        Returns the size of the pixel overlap between
        each window. This is the number of pixels added as 
        margin around each block

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getPercent(self)
        Returns the percent complete. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getPixColRow(self, x, y)
        This function is for internal use only. The user should
        be looking at getBlockCoordArrays() or getPixRowColBlock()
        for dealing with blocks and coordinates.
        
        Get the (col, row) relative to the current image grid,
        for the nominated pixel within the current block. The
        given (x, y) are column/row numbers (starting at zero),
        and the return is a tuple::
        
            (column, row)
        
        where these are relative to the whole of the current
        working grid. If working with a single raster, this is the same
        as for that raster, but if working with multiple rasters, 
        the working grid is the intersection or union of them. 
        
        Note that this function will give incorrect/misleading results
        if used in conjunction with a block overlap. 
         

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getPixRowColBlock(self, x, y)
        Return the row/column numbers, within the current block,
        for the pixel which contains the given (x, y) coordinate.
        The coordinates of (x, y) are in the world coordinate
        system of the reference grid. The row/col numbers are 
        suitable to use as array indices in the array(s) for the 
        current block. If the nominated pixel is not contained
        within the current block, the row and column numbers are
        both None (hence this should be checked). 
        
        Return value is a tuple of 2 int values
            (row, col)

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getPixelSize(self)
        Gets the current pixel size and returns it as a tuple (x and y)

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getProjection(self)
        Return the WKT describing the current
        projection system

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getTotalBlocks(self)
        Returns the total number of blocks the dataset
        has been split up into for processing

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getTotalSize(self)
        Returns the total size (in pixels) of the dataset
        being processed

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getTransform(self)
        Return the current transform between world
        and pixel coords. This is as defined by GDAL. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.getWindowSize(self)
        Returns the size of the current window. Returns a 
        tuple (numCols, numRows)

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.isFirstBlock(self)
        Returns True if this is the first block to be processed

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.isLastBlock(self)
        Returns True if this is the last block to be processed

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.setBlockBounds(self, blocktl, blockbr)
        Sets the coordinate bounds of the current block
        
        This routine is for internal use by RIOS. Its use in any other
        context is not sensible. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.setBlockCount(self, xblock, yblock)
        Sets the count of the current block
        
        This routine is for internal use by RIOS. Its use in any other
        context is not sensible. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.setBlockDataset(self, block, dataset, filename)
        Saves a match between the numpy block read
        and it's GDAL dataset. So we can look up the
        dataset later given a block.
        
        This routine is for internal use by RIOS. Its use in any other
        context is not sensible. 
        
        This is no longer implemented, and raises an exception if called.

#### &nbsp;&nbsp;&nbsp;&nbsp; ReaderInfo.setBlockSize(self, blockwidth, blockheight)
        Sets the size of the current block
        
        This routine is for internal use by RIOS. Its use in any other
        context is not sensible. 

## Functions
### def makeReaderInfo(workinggrid, blockDefn, controls, infiles, inputs, allInfo)
        Construct a ReaderInfo object for the current block.
        
        In earlier versions of RIOS, this was constructed more organically
        during the block iteration process (although it was rather obscure,
        even then). In newer versions, we are putting it together from
        other information, to maintain full compatibility when this is passed
        to the user function. It is not used for any other purpose within RIOS.

