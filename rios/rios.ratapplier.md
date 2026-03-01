# rios.ratapplier
    Apply a function to a whole Raster Attribute Table (RAT), block by block,
    so as to avoid using large amounts of memory. Transparently takes care of 
    the details of reading and writing columns from the RAT. 
    
    This was written in rough mimicry of the RIOS image applier functionality. 
    
    The most important components are the :func:`rios.ratapplier.apply` function, and 
    the :class:`rios.ratapplier.RatApplierControls` class. Pretty much everything else is for internal 
    use only. The docstring for the :func:`rios.ratapplier.apply` function gives a simple example
    of its use. 
    
    In order to work through the RAT(s) block by block, we rely on having
    available routines to read/write only a part of the RAT. This is available
    with GDAL 1.11 or later. If this is not available, we fudge the same thing 
    by reading/writing whole columns, i.e. the block size is the full length 
    of the RAT. This last case is not efficient with memory, but at least 
    provides the same functionality. 

## Classes
### class BlockCollection
      Hold a set of RatBlockAssociation objects, for all currently open RATs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BlockCollection.\_\_init\_\_(self, ratAssoc, state, allFileHandles, controls)
          Create a RatBlockAssociation entry for every RatHandle in ratAssoc

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BlockCollection.clearCache(self)
          Clear all caches

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BlockCollection.finaliseRowCount(self, outputRatHandleNameList)
          Called after the block loop completes, to reset the row count of
          each output RAT, in case it had been over-allocated. 
          
          In some raster formats, this will not reclaim space, but we still would
          like the row count to be correct. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BlockCollection.writeCache(self, outputRatHandleNameList, controls, state)
          Write all cached data blocks

### class FileHandles
      Hang onto all the required file-related objects relating to a given
      opened RAT. For a GDAL RAT, these are the GDAL objects, for a
      Zarr-based RAT, just the RatZarr object. The unused objects are None.
      
      Attributes are:
      
          * **ds**                  The gdal.Dataset object
          * **band**                The gdal.Band object
          * **gdalRat**             The gdal.RasterAttributeTable object
          * **columnNdxByName**     A lookup table to get column index from column name
          * **rz**                  The RatZarr object
          

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandles.\_\_init\_\_(self, ratHandle, update=False, sharedDS=None)
          If update is True, the GDAL dataset is opened with gdal.GA_Update.
          If sharedDS is not None, this is used as the GDAL dataset, rather
          than opening a new one. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandles.close(self)
          Close any open file handles

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandles.getRatObj(self)
          Return the RAT object. For Zarr, this is just the RatZarr object,
          while for GDAL it is the RasterAttributeTable object

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandles.getRowCount(self)
          Return the current row count of the RAT object

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandles.setRowCount(self, rowCount)
          Set the row count on the appropriate RAT object

### class FileHandlesCollection
      A set of all the FileHandles objects

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandlesCollection.\_\_init\_\_(self, inRats, outRats)
          Open all the raster files, storing a dictionary of FileHandles objects
          as self.fileHandlesDict. This is keyed by RatHandle objects. 
          
          Output files are opened first, with update=True. Any input files which
          are not open are then opened with update=False.
          
          Extra effort is made to cope with the very unlikely case of opening
          RATs on separate layers in the same image file, mostly because if
          it did happen, it would go horribly wrong. Such cases are able to share 
          the same gdal.Dataset object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandlesCollection.checkConsistency(self)
          Check the consistency of the set of input RATs opened on the current instance.
          It is kind of assumed that the output rats will become consistent, although
          this is by no means guaranteed. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandlesCollection.checkExistingDS(self, ratHandle)
          Checks the current set of filenames in use, and if it finds one with the same
          filename as the given ratHandle, assumes that it is already open, but with a 
          different layer number. If so, return the gdal.Dataset associated with it,
          so it can be shared. If not found, return None. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandlesCollection.close(self)
          Close all file handles

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileHandlesCollection.getRowCount(self)
          Return the number of rows in the RATs of all files. Actually
          just returns the row count of the first input RAT, assuming 
          that they are all the same (see self.checkConsistency())

### class OtherArguments
      Simple empty class which can be used to pass arbitrary arguments in and 
      out of the apply() function, to the user function. Anything stored on
      this object persists between iterations over blocks. 

### class RatApplierControls
      Controls object for the ratapplier. An instance of this class can
      be given to the apply() function, to control its behaviour. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierControls.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierControls.outputRowCountHandling(self, method=0, totalsize=None, incrementsize=None)
          Determine how the row count of the output RAT(s) is handled. The method
          argument can be one of the following constants:
          
              * RCM_EQUALS_INPUT        Output RAT(s) have same number of rows as input RAT(s)
              * RCM_FIXED               Output row count is set to a fixed size
              * RCM_INCREMENT           Output row count is incremented as required
              
          
          The totalsize and incrementsize arguments, if given, should be int.
          
          totalsize is used to set the output row count when the method is RCM_FIXED. 
          It is required, if the method is RCM_FIXED. 
          
          incrementsize is used to determine how much the row count is
          incremented by, if the method is RCM_INCREMENT. If not given,
          it defaults to the length of the block being written. 
          
          The most common case if the default (i.e. RCM_EQUALS_INPUT). If the
          output RAT row count will be different from the input, and the count can 
          be known in advance, then you should use RCM_FIXED to set that size. Only 
          if the output RAT row count cannot be determined in advance should 
          you use RCM_INCREMENT. 
          
          For some raster formats, using RCM_INCREMENT will result in wasted
          space, depending on the incrementsize used. Caution is recommended. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierControls.setBlockLength(self, blockLen)
          Change the number of rows used per block

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierControls.setProgress(self, progress)
          Set the progress display object. Default is no progress
          object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierControls.setRowCount(self, rowCount)
          Set the total number of rows to be processed. This is normally only useful
          when doing something like writing an output RAT without any input RAT,
          so the number of rows is otherwise undefined. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierControls.setUseStringDType(self, useStringDType)
          Set whether to use the numpy-2.x StringDType when reading GFT_String
          columns. If this is True, then when data is read from a GFT_String
          column, it will be converted to StringDType (i.e. an array of
          variable-length strings) before presenting it to the user.
          
          The default is the old behaviour, i.e. the returned string arrays
          are fixed-width bytes string arrays.
          
          If StringDType is unavailable (numpy < 2.0), this flag is
          always False.

### class RatApplierState
      Current state of RAT applier. An instance of this class is passed as the first
      argument to the user function. 
      
      Attributes:
      
          * blockNdx                Index number of current block (first block is zero, second block is 1, ...)
          * startrow                RAT row number of first row of current block (first row is zero)
          * blockLen                Number of rows in current block
          * inputRowNumbers         Row numbers in whole input RAT(s) corresponding to current block
          * rowCount                The total number of rows in the input RAT(s)
          

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierState.\_\_init\_\_(self, rowCount)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatApplierState.setBlock(self, i, requestedBlockLen)
          Sets the state to be pointing at the i-th block. i starts at zero. 

### class RatAssociations
      Class associating external raster attribute tables with internal names. 
      Each attribute defined on this object should be a RatHandle object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatAssociations.getRatList(self)
          Return a list of the names of the RatHandle objects defined on this object

### class RatBlockAssociation
      Hold one or more blocks of data from RAT columns of a single RAT. This
      class is kind of at the heart of the module. 
      
      Most generic attributes on this class are blocks of data read from and
      written to the RAT, and so are not actually attributes at all, but are 
      managed by the __setattr__/__getattr__ over-ride methods. Their names are
      the names of the columns to which they correspond. However, there are a 
      number of genuine attributes which also need to be present, for internal 
      use, and it is obviously important that their names not be the same as 
      any columns. Since we obviously cannot guarantee this, we have named them 
      beginning with "Z\_\_", in the hope that no-one ever has a column with
      a name like this. These are all created within the __init__ method. 
      
      The main purpose of using __getattr__ is to avoid reading columns which 
      the userFunc is not actually using. As a consequence, one also needs to
      use __setattr__ to handle the data the same way. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.\_\_init\_\_(self, state, fileHandles, controls)
          Pass in the RatApplierState object, so we can always see where we 
          are up to, and the associated FileHandles object, so we can get to 
          the file.
          
          Note the use of object.__setattr__() to create the normal attributes
          on the object, so they do not behave as RAT column blocks. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.checkZarrfileParams(self)
          Check if the Zarr file has only just been created. If so,
          initialize it with suitable parameters

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.clearCache(self)
          Clear the current cache of data blocks

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.finaliseRowCount(self)
          If the row count for this RAT has been over-allocated, reset it back
          to the actual number of rows we wrote. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.getUsage(self, columnName)
          Return the usage of the given column

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.guessNewRowCount(self, rowsToWrite, controls, state)
          When we are writing to a new RAT, and we find that we need to write more
          rows than it currently has, we guess what we should set the row count to
          be, depending on how the controls have told us to do this. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.setUsage(self, columnName, usage)
          Set the usage of the given column. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatBlockAssociation.writeCache(self, controls, state)
          Write all cached data blocks. Creates the columns if they do not already exist. 

### class RatHandle
      A handle onto the RAT for a single image layer. This is used as an 
      easy way for the user to nominate both a filename and a layer number. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatHandle.\_\_init\_\_(self, filename, layernum=1)
          This is how the user specifies a RAT stored in a GDAL file.
          
          filename is a string, layernum is an integer (first layer is 1)

### class RatZarrHandle
      Equivalent of RatHandle, but for a RAT stored in a Zarr file
      
      New in version 2.0.9

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarrHandle.\_\_init\_\_(self, filename)
          This is how the user specifies a RAT stored in a Zarr file.
          
          filename is a string. For local files, should be just a path string,
          for AWS S3 files it should have the form s3://bucket/path
          
          New in version 2.0.9

## Functions
### def apply(userFunc, inRats, outRats, otherargs=None, controls=None)
        Apply the given function across the whole of the given raster attribute tables.
        The attribute table is processing one chunk at a time allowing very large tables
        without running out of memory.
        
        All raster files must already exist, but new columns can be created. 
        
        Normal pattern is something like the following::
        
            inRats = ratapplier.RatAssociations()
            outRats = ratapplier.RatAssociations()
            
            inRats.vegclass = ratapplier.RatHandle('vegclass.kea')
            outRats.vegclass = ratapplier.RatHandle('vegclass.kea')
            
            ratapplier.apply(myFunc, inRats, outRats)
            
            def myFunc(info, inputs, outputs):
                outputs.vegclass.colSum = inputs.vegclass.col1 + inputs.vegclass.col2
        
        The :class:`rios.ratapplier.RatHandle` defaults to using the RAT from the first layer of 
        the image which is usual for thematic imagery. 
        This can be overridden using the layernum parameter. The names of the columns are reflected in 
        the names of the fields on the inputs and outputs parameters and multiple input and output RAT's can be specified
        
        The otherargs argument can be any object, and is typically an instance
        of :class:`rios.ratapplier.OtherArguments`. It will be passed in to each call of the user function, 
        unchanged between calls, so that other values can be passed in, and 
        calculated quantities passed back. The values stored on this object are not
        directly associated with rows of the RAT, and must be managed entirely by
        the user. If it is not required, it need not be passed. 
        
        The controls object is an instance of the :class:`rios.ratapplier.RatApplierControls` class, and 
        is only required if the default control settings are to be changed. 
        
        The info object which is passed to the user function is an instance of 
        the :class:`rios.ratapplier.RatApplierState` class. 
        
        By default new columns are marked as 'Generic'. If they need to be marked as having a specific usage, the following syntax is used::
        
            def addCols(info, inputs, outputs):
                "Add two columns and output"
                outputs.outimg.colSum = inputs.inimg.col1 + inputs.inimg.col4
                outputs.outImg.Red = someRedValue      # some calculated red value, in 0-255 range
                outputs.outImg.setUsage('Red', gdal.GFU_Red)
        
        **Statistics**
        
        Since the RAT is now read one chunk at a time calling numpy functions like mean() 
        etc will only return statistics for the current chunk, not globally. The solution is to use the 
        :class:`rios.fileinfo.RatStats` class::
        
            from rios.fileinfo import RatStats
        
            columnsOfInterest = ['col1', 'col4']
            ratStatsObj = RatStats('file.img', columnlist=columnsOfInterest)
        
            print(ratStatsObj.col1.mean, ratStatsObj.col4.mean)
        
        Each column attribute is an instance of :class:`rios.fileinfo.ColumnStats` and is intended to be 
        passed through the apply function via the otherargs mechanism.
        
        **Non-GDAL RATs**
        
        An alternative form of RAT is supported, based on Zarr arrays. Instead of
        using the RatHandle class to connect to a RAT in a GDAL file, use the
        RatZarrHandle class to associate with a Zarr file. This has a very specific
        internal structure, and is intended for use in writing columns outside
        of the GDAL raster file, if it cannot be written for some reason. The main
        use case is to read and/or write a RAT stored on AWS S3, but it also works
        the same if the file is on local disk.
        
        Example::
        
            inRats.vegclass = ratapplier.RatHandle('vegclass.kea')
            outRats.extra = ratapplier.RatZarrHandle('s3://mybucket/extra.zarr')
        
        This can then used in the same way as a GDAL-based RAT.
        
        It requires the ratzarr package (https://github.com/ubarsc/ratzarr).

### def copyRAT(infile, outfile, progress=None, omitColumns=None)
        Given an input and output filenames copies the RAT from 
        the input and writes it to the output.
        
        If omitColumns is set, then it should be a sequence of
        columns names that are to be omitted from the copying.
        For example, the 'Histogram' column may need to be omitted
        so that the pixel counts stay the correct values in the output
        image.
        
        infile and outfile can both be RatZarr files. If the output file
        does not exist, it will be created as a RatZarr file.

### def internalCopyRAT(info, inputs, outputs, otherArgs)
        Called from copyRAT. Copies the RAT

## Values
    RCM_EQUALS_INPUT = 0
    RCM_FIXED = 1
    RCM_INCREMENT = 2
    division = _Feature((2, 2, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 131072)
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
    ratzarr = None
