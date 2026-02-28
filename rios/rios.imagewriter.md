# rios.imagewriter
    Contains functions used to write output files from applier.apply.
    
    Also contains the now-deprecated ImageWriter class

## Classes
### class ImageWriter
      This class is the opposite of the ImageReader class and is designed
      to be used in conjunction. The easiest way to use it is pass the
      info returned by the ImageReader for first iteration to the constructor.
      Otherwise, image size etc must be passed in.
      
      The write() method can be used to write a block (numpy array)at a 
      time to the output image - this is designed to be used at each 
      iteration through the ImageReader object. Otherwise, the writeAt() 
      method can be used to write blocks to arbitary locations.
      
      **Example**
      
      ::
      
          import sys
          from rios.imagereader import ImageReader
          from rios.imagewriter import ImageWriter
      
          inputs = [sys.argv[1],sys.argv[2]]
          reader = ImageReader(inputs) 
          writer = None 
          for (info, blocks) in reader:     
              block1,block2 = blocks
              out = block1 * 4 + block2
              if writer is None:
                  writer = ImageWriter(sys.argv[3],info=info,
                      firstblock=out)
              else:
                  writer.write(out)
      
          writer.close(calcStats=True)

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.\_\_init\_\_(self, filename, drivername='HFA', creationoptions=None, nbands=None, gdaldatatype=None, firstblock=None, info=None, xsize=None, ysize=None, transform=None, projection=None, windowxsize=None, windowysize=None, overlap=None)
        filename is the output file to be created. Set driver to name of
        GDAL driver, default it HFA. creationoptions will also need to be
        set if not using HFA since the default probably does not make sense
        for other drivers.
        
        Either pass nbands and gdaldatatype OR firstblock. If you pass 
        firstblock, nbands and gdaldataype will be determined from that block
        and that block written to file.
        
        Also, either pass info (the first argument returned from each iteration
        through ImageReader, generally create this class on the first iteration)
        or xsize, ysize, transform, projection, windowxsize, windowysize and overlap
        If you pass info, these other values will be determined from that

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.addAutoColorTable(self, autoColorTableType)
        If autoColorTable has been set up for this output, then generate
        a color table of the requested type, and add it to the current
        file. This is called AFTER the Dataset has been closed, so is performed on
        the filename. This only applies to thematic layers, so when we open the file
        and find that the layers are athematic, we do nothing. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.close(self, calcStats=False, statsIgnore=None, progress=None, omitPyramids=False, overviewLevels=[4, 8, 16, 32, 64, 128, 256, 512], overviewMinDim=128, overviewAggType=None, autoColorTableType=None, approx_ok=False)
        Closes the open dataset

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.deleteIfExisting(filename)
        Delete the filename if it already exists.
        If possible, use the appropriate GDAL driver to do so, to ensure
        that any associated files will also be deleted. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.doubleCheckCreationOptions(self, drivername, creationoptions)
        Try to ensure that the given creation options are not incompatible with RIOS
        operations. Does not attempt to ensure they are totally valid, as that is GDAL's
        job. 
        
        Returns a copy of creationoptions, possibly modified, or raises ImageOpenError
        in cases where the problem is not fixable. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.getCurrentBlock(self)
        Returns the number of the current block

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.getGDALDataset(self)
        Returns the underlying GDAL dataset object

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.reset(self)
        Resets the location pointer so that the next
        write() call writes to the start of the file again

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.setLayerNames(self, names)
        Sets the output layer names. Pass a list
        of layer names, one for each output band

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.setThematic(self)
        Sets the output file to thematic. If file is multi-layer,
        then all bands are set to thematic. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.write(self, block)
        Writes the numpy block to the current location in the file,
        and updates the location pointer for next write

#### &nbsp;&nbsp;&nbsp;&nbsp; ImageWriter.writeAt(self, block, xcoord, ycoord)
        writes the numpy block to the specified pixel coords
        in the file

## Functions
### def addAutoColorTable(filename, autoColorTableType)
        If autoColorTable has been set up for this output, then generate
        a color table of the requested type, and add it to the current
        file. This is called AFTER the Dataset has been closed, so is performed on
        the filename. This only applies to thematic layers, so when we open the file
        and find that the layers are athematic, we do nothing. 

### def allnotNone(items)

### def anynotNone(items)

### def closeOutfiles(gdalOutObjCache, outfiles, controls, singlePassMgr, timings)
        Close all the output files

### def deleteIfExisting(filename)
        Delete the filename if it already exists. If possible, use the
        appropriate GDAL driver to do so, to ensure that any associated
        files will also be deleted.

### def doubleCheckCreationOptions(drivername, creationoptions, controls, workinggrid)
        Try to ensure that the given creation options are compatible with
        RIOS operations. Does not attempt to ensure they are totally valid, as
        that is GDAL's job.
        
        If it finds any incompatibility, an exception is raised.

### def makeMinMaxList(singlePassMgr, symbolicName, seqNum)
        Make a list of min/max values per band, for the nominated output file,
        from values already present on singlePassMgr.
        Mimicing the list returned by addBasicStatsGDAL, for use with
        addHistogramsGDAL.

### def openOutfile(symbolicName, filename, controls, arr, workinggrid)
        Open the requested output file

### def setDefaultDriver()
        Sets some default values into global variables, defining
        what defaults we should use for GDAL driver. On any given
        output file these can be over-ridden, and can be over-ridden globally
        using the environment variables
        
            * $RIOS_DFLT_DRIVER
            * $RIOS_DFLT_DRIVEROPTIONS
            * $RIOS_DFLT_CREOPT_<drivername>
        
        If RIOS_DFLT_DRIVER is set, then it should be a gdal short driver name. 
        If RIOS_DFLT_DRIVEROPTIONS is set, it should be a space-separated list
        of driver creation options, e.g. "COMPRESS=LZW TILED=YES", and should
        be appropriate for the selected GDAL driver. This can also be 'None'
        in which case an empty list of creation options is passed to the driver.
        
        The same rules apply to the driver-specific creation options given
        using $RIOS_DFLT_CREOPT_<driver>. These options are a later paradigm, and 
        are intended to supercede the previous generic driver defaults. 
        
        If not otherwise supplied, the default is to use the HFA driver, with compression. 
        
        The code here is more complex than desirable, because it copes with legacy behaviour
        in the absence of the environment variables, and in the absence of the driver-specific
        option variables. 
            

### def writeBlock(gdalOutObjCache, blockDefn, outfiles, outputs, controls, workinggrid, singlePassMgr, timings)
        Write the given block to the files given in outfiles

## Values
    DEFAULTCREATIONOPTIONS = ['COMPRESSED=TRUE', 'IGNOREUTM=TRUE']
    DEFAULTDRIVERNAME = 'HFA'
    dfltDriverOptions = {'HFA': ['COMPRESSED=TRUE', 'IGNOREUTM=TRUE'], 'GTiff': ['TILED=YES', 'COMPRESS=LZW', 'INTERLEAVE=BAND', 'BIGTIFF=IF_SAFER']}
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
