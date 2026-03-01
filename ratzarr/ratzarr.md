# ratzarr
    Routines for handling RAT-like columns stored in a Zarr array group.
    
    The raster data model in GDAL includes a Raster Attribute Table (RAT).
    Several formats support this, storing the data inside the same raster file.
    However, sometimes it is not possible to write back into the file, such as
    when the file is stored on AWS S3 storage.
    
    This ratzarr module provides an alternative way to store RAT-like columns
    alongside the raster file itself, in a way that allows extra columns to
    be written incrementally (i.e. block-by-block), allowing for very large
    tables to be built up, both in terms of large row count and large number of
    columns. More columns can be added later, as required, and columns can be
    deleted individually. Columns can also be resized to a new row count.
    
    The RAT is stored as a set of 1-d arrays in a single Zarr array group
    (https://zarr.dev/, https://github.com/zarr-developers/zarr-python). Currently,
    ratzarr only makes use of the local disk and AWS S3 storage options. It does
    not support zipfile storage, as this does not allow for writeable arrays on S3.
    
    Simple usage:
    
        import ratzarr
    
        ratfile = 's3://mybucket/somepath/myrat.zarr'
        nRows = 1000000
        rz = ratzarr.RatZarr(ratfile)
        rz.setRowCount(nRows)
    
        # A very boring column
        col = numpy.arange(nRows) + 10
    
        rz.createColumn('BoringCol', col.dtype)
        rz.writeBlock('BoringCol', col, 0)
    
        # Read it all back
        col2 = rz.readBlock('BoringCol', 0, nRows)
    
    For the RAT to be on local disk, the ``ratfile`` should just be an ordinary
    path string.
    
    For very large RATs, with large numbers of rows and possibly many columns,
    it is often more memory-efficient to work with fixed-size blocks of data.
    In this case, it is important to pay attention to the Zarr chunk size. The
    default value is 500000, and it can be set before any columns are created
    (see RatZarr.setChunkSize). Most importantly, the size of blocks being read
    and/or written should be divisible by the chunk size. The simplest way is
    often just to choose the block size as a multiple of the default Zarr chunk
    size, but other considerations may require directly setting the Zarr chunk
    size.
    
    It is intended that all columns have the same length (i.e. number of rows), but
    currently it is up to the user to enforce this.

## Classes
### class AllTests(TestCase)
      Run all tests

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.\_\_init\_\_(self, methodName='runTest')
          Create an instance of the class that will use the named test
          method when executed. Raises a ValueError if the instance does
          not have a method with the specified name.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.deleteTestFile(self, filename)
          Delete the given test file, if it exists

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.makeFilename(self, filename)
          Make full filename string

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_chunksize(self)
          Chunk size manipulation

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_colnames(self)
          Handling column names

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_flags(self)
          Test a bunch of exception conditions on constructor flags

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_resize(self)
          Reset rowCount

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_simple(self)
          Test reading/writing/creating a simple RAT

### class RatZarr

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.\_\_init\_\_(self, filename, readOnly=False, create=True)
          This class is a rough equivalent of the GDAL RasterAttributeTable
          class, implementing a RAT-like structure with a Zarr group.
          It is not supposed to be a drop-in replacement, just something with
          somewhat similar functionality.
          
          Parameters
          ----------
            filename : str
              The Zarr file/store to use for the RAT. Can be a local
              path string, or a URL for S3, e.g. 's3://bucketname/path'
            readOnly : bool
              If True, the RAT cannot be modified
            create : bool
              If True (the default), the RAT will be created if it does not
              already exist
          
          The type of each column is a numpy dtype. We do not reproduce GDAL's
          GFT_Integer/Float/String field types, mainly because they seemed to
          serve no particular purpose in this context. All the usual numpy
          integer and float dtypes are supported, but Zarr only handles string
          arrays if they have dtype `numpy.dtypes.StringDType()`.
          
          This sort of RAT should not be used for storing things like the
          histogram or colour table. For this reason, we do not reproduce the
          concept of column usage, and all columns are effectively GFU_Generic.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.colExists(self, colName)
          Return True if the column already exists
          
          Parameters
          ----------
            colName : str
              Name of column

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.createColumn(self, colName, dtype)
          Create a new column. Uses the currently active rowCount
          
          Parameters
          ----------
            colName : str
              Name of column
            dtype : Any numpy dtype
              The data type of the column to create

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.delete(zarrfile)
          Delete the named RatZarr file.
          
          Silently returns if file does not exists. Raises exception if
          the file is not a valid RatZarr file.
          
          Parameters
          ----------
            zarrfile : str
              Full name of a possible zarr file (including 's3://' if required)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.deleteColumn(self, colName)
          Delete the named column
          
          Parameters
          ----------
            colName : str
              Name of column

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.exists(zarrfile)
          Check if the given filename exists. Does not confirm if it is a
          valid Zarr or RatZarr file.
          
          Parameters
          ----------
            zarrfile : str
              Full name of a possible zarr file (including 's3://' if required)
          
          Returns
          -------
            exists : bool
              True if the named file exists

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getColumnAttributeMapping(self, colName)
          Return the column attributes mapping of the given column.
          
          The returned object is a mapping (i.e. behaves roughly like a
          dictionary). It is directly connected to the Zarr file, and as such,
          any changes made are reflected immediately on disk. Some attributes
          are already present, managed by RatZarr, but others can be added to
          suit the user.
          
          The keys should be hashable and JSON-serializable, and you probably
          should stick to using strings only. The values can be anything
          JSON-serializable, so should largely be limited to int, float, str,
          list, dict.
          
          Parameters
          ----------
            colName : str
              Name of column
          
          Returns
          -------
            attrs : zarr.core.attributes.Attributes
              The Zarr attributes mapping attached to the requested column

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getColumnChunkSize(self, colName)
          Get the chunk size for the given column. Normally this is the same as
          the chunk size for the whole RAT, but if you suspect it is different,
          use this to check.
          
          Parameters
          ----------
            colName : str
              Name of column
          
          Returns
          -------
            chunkSize : int
              The chunk size for the given column (usually set when it was
              created)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getColumnNames(self)
          Returns
          -------
            colNameList : list of str
              List of column names in RAT

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getModificationLog(self, colName)
          Read the modification log for the given column
          
          The log is a list of tuples. Each tuple is (dt, user, argv), where
          dt is a datetime.datetime of the modification event, user is
          a string of the username running the command, and argv is the
          command (list of str) being run to make the modification.
          
          A modification event is when writeBlock() is called. For a given
          instance of RatZarr, only the first writeBlock for each column
          is recorded, so in general only one event per column is recorded
          for a given program run.
          
          Parameters
          ----------
            colName : str
              Name of column
          
          Returns
          -------
            modLog : list of tuple
              List of modification events for the column.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getRATChunkSize(self)
          Return the chunk size for the RAT.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getRowCount(self)
          Returns
          -------
            rowCount : int
              The current number of rows in the RAT

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.isValidRatZarr(zarrfile)
          Check if the given filename is a valid RatZarr file
          
          Parameters
          ----------
            zarrfile : str
              Full name of a possible zarr file (including 's3://' if required)
          
          Returns
          -------
            isValid : bool
              True if the named file exists and is valid RatZarr

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.openColumn(self, colName)
          Open a cached connection to an existing array on disk.
          
          This will happen automatically when reading or writing, and users
          should not usually need to call this method.
          
          Parameters
          ----------
            colName : str
              Name of column

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.readBlock(self, colName, startRow, blockLen)
          Read a block from the given column. Return a numpy array of the
          block.
          
          To read the entire column, use startRow=0 and blockLen=rowCount.
          
          For best efficiency, use a block length which is a multiple of the
          chunk size (see getRATChunkSize, getColumnChunkSize).
          
          If startRow+blockLen is larger than the column length, only
          the available rows are read, and the returned array has this smaller
          size.
          
          Parameters
          ----------
            colName : str
              Name of column
            startRow : int
              Row of first element to read (starts at 0)
            blockLen : int
              Number of rows to read
          
          Returns
          -------
            block : ndarray
              1-d numpy array of data for block

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.setChunkSize(self, chunksize)
          Set the Zarr chunk size for all columns created after this call.
          This defaults to a sensible value, and should only be changed if
          you know what you are doing. Chunk sizes either too large or too
          small can have performance implications, so proceed with caution.
          
          The chunk size is preserved in the disk file, and will apply when
          next it is opened. Usually best to set this once, so that all
          columns have the same chunk size.
          
          When reading and/or writing block-by-block, it is strongly recommended
          that the block length be a multiple of the chunk size, to avoid
          significant performance degradation.
          
          Parameters
          ----------
            chunksize : int
              Number of rows per chunk

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.setRowCount(self, rowCount)
          Set the number of rows in the RAT.
          
          Usually do this before creating any columns. The row count persists
          between runs, so this only needs to be set when the RAT is first
          created, or if it needs to be changed.
          
          If there are existing columns, they will be resized to the new
          rowCount. If this less than current, existing columns will be
          truncated.
          
          Parameters
          ----------
            rowCount : int
              The desired number of rows in the RAT

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RatZarr.writeBlock(self, colName, block, startRow)
          Write the given block of data into the named column, beginning at
          startRow. For best performance, use a block length which is a
          multiple of the chunk size (see setChunkSize, getRATChunkSize,
          getColumnChunkSize).
          
          Parameters
          ----------
            colName : str
              Name of column
            block : ndarray
              1-d numpy array
            startRow : int
              Row of first element of block (starts at 0)

### class RatZarrError(Exception)
      Generic exception for RatZarr

## Functions
### def mainCmd()

## Values
    CHUNKSIZE_ATTR = 'CHUNKSIZE'
    DFLT_CHUNKSIZE = 500000
    MODIFICATION_ATTR = 'MODIFICATIONLOG'
    boto3 = None
    colNameByType = {<class 'numpy.int32'>: 'int32col', <class 'numpy.float32'>: 'float32col', StringDType(): 'stringcol'}
