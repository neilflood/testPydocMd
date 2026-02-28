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
### class AllTests
      Run all tests

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.\_\_init\_\_(self, methodName='runTest')
        Create an instance of the class that will use the named test
        method when executed. Raises a ValueError if the instance does
        not have a method with the specified name.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._addExpectedFailure(self, result, exc_info)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._addUnexpectedSuccess(self, result)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._baseAssertEqual(self, first, second, msg=None)
        The default assertEqual implementation, not type specific.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._callCleanup(self, function, /, *args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._callSetUp(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._callTearDown(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._callTestMethod(self, method)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._deprecate(original_func)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._formatMessage(self, msg, standardMsg)
        Honour the longMessage attribute when generating failure messages.
        If longMessage is False this means:
        * Use only an explicit message if it is provided
        * Otherwise use the standard message for the assert
        
        If longMessage is True:
        * Use the standard message
        * If an explicit message is provided, plus ' : ' and the explicit message

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._getAssertEqualityFunc(self, first, second)
        Get a detailed comparison function for the types of the two args.
        
        Returns: A callable accepting (first, second, msg=None) that will
        raise a failure exception if first != second with a useful human
        readable error message for those types.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests._truncateMessage(self, message, diff)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.addClassCleanup(function, /, *args, **kwargs)
        Same as addCleanup, except the cleanup items are called even if
        setUpClass fails (unlike tearDownClass).

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.addCleanup(self, function, /, *args, **kwargs)
        Add a function, with arguments, to be called when the test is
        completed. Functions added are called on a LIFO basis and are
        called after tearDown on test failure or success.
        
        Cleanup items are called even if setUp fails (unlike tearDown).

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.addTypeEqualityFunc(self, typeobj, function)
        Add a type specific assertEqual style function to compare a type.
        
        This method is for use by TestCase subclasses that need to register
        their own type equality functions to provide nicer error messages.
        
        Args:
            typeobj: The data type to call this function on when both values
                    are of the same type in assertEqual().
            function: The callable taking two arguments and an optional
                    msg= argument that raises self.failureException with a
                    useful error message when the two arguments are not equal.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertAlmostEqual(self, first, second, places=None, msg=None, delta=None)
        Fail if the two objects are unequal as determined by their
        difference rounded to the given number of decimal places
        (default 7) and comparing to zero, or by comparing that the
        difference between the two objects is more than the given
        delta.
        
        Note that decimal places (from zero) are usually not the same
        as significant digits (measured from the most significant digit).
        
        If the two objects compare equal then they will automatically
        compare almost equal.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertCountEqual(self, first, second, msg=None)
        Asserts that two iterables have the same elements, the same number of
        times, without regard to order.
        
            self.assertEqual(Counter(list(first)),
                             Counter(list(second)))
        
         Example:
            - [0, 1, 1] and [1, 0, 1] compare equal.
            - [0, 0, 1] and [0, 1] compare unequal.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertDictContainsSubset(self, subset, dictionary, msg=None)
        Checks whether dictionary is a superset of subset.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertDictEqual(self, d1, d2, msg=None)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertEqual(self, first, second, msg=None)
        Fail if the two objects are unequal as determined by the '=='
        operator.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertFalse(self, expr, msg=None)
        Check that the expression is false.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertGreater(self, a, b, msg=None)
        Just like self.assertTrue(a > b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertGreaterEqual(self, a, b, msg=None)
        Just like self.assertTrue(a >= b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertIn(self, member, container, msg=None)
        Just like self.assertTrue(a in b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertIs(self, expr1, expr2, msg=None)
        Just like self.assertTrue(a is b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertIsInstance(self, obj, cls, msg=None)
        Same as self.assertTrue(isinstance(obj, cls)), with a nicer
        default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertIsNone(self, obj, msg=None)
        Same as self.assertTrue(obj is None), with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertIsNot(self, expr1, expr2, msg=None)
        Just like self.assertTrue(a is not b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertIsNotNone(self, obj, msg=None)
        Included for symmetry with assertIsNone.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertLess(self, a, b, msg=None)
        Just like self.assertTrue(a < b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertLessEqual(self, a, b, msg=None)
        Just like self.assertTrue(a <= b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertListEqual(self, list1, list2, msg=None)
        A list-specific equality assertion.
        
        Args:
            list1: The first list to compare.
            list2: The second list to compare.
            msg: Optional message to use on failure instead of a list of
                    differences.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertLogs(self, logger=None, level=None)
        Fail unless a log message of level *level* or higher is emitted
        on *logger_name* or its children.  If omitted, *level* defaults to
        INFO and *logger* defaults to the root logger.
        
        This method must be used as a context manager, and will yield
        a recording object with two attributes: `output` and `records`.
        At the end of the context manager, the `output` attribute will
        be a list of the matching formatted log messages and the
        `records` attribute will be a list of the corresponding LogRecord
        objects.
        
        Example::
        
            with self.assertLogs('foo', level='INFO') as cm:
                logging.getLogger('foo').info('first message')
                logging.getLogger('foo.bar').error('second message')
            self.assertEqual(cm.output, ['INFO:foo:first message',
                                         'ERROR:foo.bar:second message'])

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertMultiLineEqual(self, first, second, msg=None)
        Assert that two multi-line strings are equal.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertNoLogs(self, logger=None, level=None)
        Fail unless no log messages of level *level* or higher are emitted
        on *logger_name* or its children.
        
        This method must be used as a context manager.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertNotAlmostEqual(self, first, second, places=None, msg=None, delta=None)
        Fail if the two objects are equal as determined by their
        difference rounded to the given number of decimal places
        (default 7) and comparing to zero, or by comparing that the
        difference between the two objects is less than the given delta.
        
        Note that decimal places (from zero) are usually not the same
        as significant digits (measured from the most significant digit).
        
        Objects that are equal automatically fail.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertNotEqual(self, first, second, msg=None)
        Fail if the two objects are equal as determined by the '!='
        operator.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertNotIn(self, member, container, msg=None)
        Just like self.assertTrue(a not in b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertNotIsInstance(self, obj, cls, msg=None)
        Included for symmetry with assertIsInstance.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertNotRegex(self, text, unexpected_regex, msg=None)
        Fail the test if the text matches the regular expression.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertRaises(self, expected_exception, *args, **kwargs)
        Fail unless an exception of class expected_exception is raised
        by the callable when invoked with specified positional and
        keyword arguments. If a different type of exception is
        raised, it will not be caught, and the test case will be
        deemed to have suffered an error, exactly as for an
        unexpected exception.
        
        If called with the callable and arguments omitted, will return a
        context object used like this::
        
             with self.assertRaises(SomeException):
                 do_something()
        
        An optional keyword argument 'msg' can be provided when assertRaises
        is used as a context object.
        
        The context manager keeps a reference to the exception as
        the 'exception' attribute. This allows you to inspect the
        exception after the assertion::
        
            with self.assertRaises(SomeException) as cm:
                do_something()
            the_exception = cm.exception
            self.assertEqual(the_exception.error_code, 3)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertRaisesRegex(self, expected_exception, expected_regex, *args, **kwargs)
        Asserts that the message in a raised exception matches a regex.
        
        Args:
            expected_exception: Exception class expected to be raised.
            expected_regex: Regex (re.Pattern object or string) expected
                    to be found in error message.
            args: Function to be called and extra positional args.
            kwargs: Extra kwargs.
            msg: Optional message used in case of failure. Can only be used
                    when assertRaisesRegex is used as a context manager.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertRegex(self, text, expected_regex, msg=None)
        Fail the test unless the text matches the regular expression.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertSequenceEqual(self, seq1, seq2, msg=None, seq_type=None)
        An equality assertion for ordered sequences (like lists and tuples).
        
        For the purposes of this function, a valid ordered sequence type is one
        which can be indexed, has a length, and has an equality operator.
        
        Args:
            seq1: The first sequence to compare.
            seq2: The second sequence to compare.
            seq_type: The expected datatype of the sequences, or None if no
                    datatype should be enforced.
            msg: Optional message to use on failure instead of a list of
                    differences.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertSetEqual(self, set1, set2, msg=None)
        A set-specific equality assertion.
        
        Args:
            set1: The first set to compare.
            set2: The second set to compare.
            msg: Optional message to use on failure instead of a list of
                    differences.
        
        assertSetEqual uses ducktyping to support different types of sets, and
        is optimized for sets specifically (parameters must support a
        difference method).

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertTrue(self, expr, msg=None)
        Check that the expression is true.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertTupleEqual(self, tuple1, tuple2, msg=None)
        A tuple-specific equality assertion.
        
        Args:
            tuple1: The first tuple to compare.
            tuple2: The second tuple to compare.
            msg: Optional message to use on failure instead of a list of
                    differences.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertWarns(self, expected_warning, *args, **kwargs)
        Fail unless a warning of class warnClass is triggered
        by the callable when invoked with specified positional and
        keyword arguments.  If a different type of warning is
        triggered, it will not be handled: depending on the other
        warning filtering rules in effect, it might be silenced, printed
        out, or raised as an exception.
        
        If called with the callable and arguments omitted, will return a
        context object used like this::
        
             with self.assertWarns(SomeWarning):
                 do_something()
        
        An optional keyword argument 'msg' can be provided when assertWarns
        is used as a context object.
        
        The context manager keeps a reference to the first matching
        warning as the 'warning' attribute; similarly, the 'filename'
        and 'lineno' attributes give you information about the line
        of Python code from which the warning was triggered.
        This allows you to inspect the warning after the assertion::
        
            with self.assertWarns(SomeWarning) as cm:
                do_something()
            the_warning = cm.warning
            self.assertEqual(the_warning.some_attribute, 147)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.assertWarnsRegex(self, expected_warning, expected_regex, *args, **kwargs)
        Asserts that the message in a triggered warning matches a regexp.
        Basic functioning is similar to assertWarns() with the addition
        that only warnings whose messages also match the regular expression
        are considered successful matches.
        
        Args:
            expected_warning: Warning class expected to be triggered.
            expected_regex: Regex (re.Pattern object or string) expected
                    to be found in error message.
            args: Function to be called and extra positional args.
            kwargs: Extra kwargs.
            msg: Optional message used in case of failure. Can only be used
                    when assertWarnsRegex is used as a context manager.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.countTestCases(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.debug(self)
        Run the test without collecting errors in a TestResult

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.defaultTestResult(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deleteTestFile(self, filename)
        Delete the given test file, if it exists

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.doClassCleanups()
        Execute all class cleanup functions. Normally called for you after
        tearDownClass.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.doCleanups(self)
        Execute all cleanup functions. Normally called for you after
        tearDown.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.enterClassContext(cm)
        Same as enterContext, but class-wide.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.enterContext(self, cm)
        Enters the supplied context manager.
        
        If successful, also adds its __exit__ method as a cleanup
        function and returns the result of the __enter__ method.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.fail(self, msg=None)
        Fail immediately, with the given message.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.id(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.makeFilename(self, filename)
        Make full filename string

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.run(self, result=None)

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.setUp(self)
        Hook method for setting up the test fixture before exercising it.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.setUpClass()
        Hook method for setting up class fixture before running tests in the class.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.shortDescription(self)
        Returns a one-line description of the test, or None if no
        description has been provided.
        
        The default implementation of this method returns the first line of
        the specified test method's docstring.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.skipTest(self, reason)
        Skip this test.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.subTest(self, msg=<object object at 0x7ee764ae86d0>, **params)
        Return a context manager that will return the enclosed block
        of code in a subtest identified by the optional message and
        keyword parameters.  A failure in the subtest marks the test
        case as failed but resumes execution at the end of the enclosed
        block, allowing further test code to be executed.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.tearDown(self)
        Hook method for deconstructing the test fixture after testing it.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.tearDownClass()
        Hook method for deconstructing the class fixture after running all tests in the class.

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_chunksize(self)
        Chunk size manipulation

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_colnames(self)
        Handling column names

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_flags(self)
        Test a bunch of exception conditions on constructor flags

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_resize(self)
        Reset rowCount

#### &nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_simple(self)
        Test reading/writing/creating a simple RAT

### class RatZarr

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.\_\_init\_\_(self, filename, readOnly=False, create=True)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.colExists(self, colName)
        Return True if the column already exists
        
        Parameters
        ----------
          colName : str
            Name of column

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.createColumn(self, colName, dtype)
        Create a new column. Uses the currently active rowCount
        
        Parameters
        ----------
          colName : str
            Name of column
          dtype : Any numpy dtype
            The data type of the column to create

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.delete(zarrfile)
        Delete the named RatZarr file.
        
        Silently returns if file does not exists. Raises exception if
        the file is not a valid RatZarr file.
        
        Parameters
        ----------
          zarrfile : str
            Full name of a possible zarr file (including 's3://' if required)

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.deleteColumn(self, colName)
        Delete the named column
        
        Parameters
        ----------
          colName : str
            Name of column

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.exists(zarrfile)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getColumnAttributeMapping(self, colName)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getColumnChunkSize(self, colName)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getColumnNames(self)
        Returns
        -------
          colNameList : list of str
            List of column names in RAT

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getModificationLog(self, colName)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getRATChunkSize(self)
        Return the chunk size for the RAT.

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.getRowCount(self)
        Returns
        -------
          rowCount : int
            The current number of rows in the RAT

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.isValidRatZarr(zarrfile)
        Check if the given filename is a valid RatZarr file
        
        Parameters
        ----------
          zarrfile : str
            Full name of a possible zarr file (including 's3://' if required)
        
        Returns
        -------
          isValid : bool
            True if the named file exists and is valid RatZarr

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.openColumn(self, colName)
        Open a cached connection to an existing array on disk.
        
        This will happen automatically when reading or writing, and users
        should not usually need to call this method.
        
        Parameters
        ----------
          colName : str
            Name of column

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.readBlock(self, colName, startRow, blockLen)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.setChunkSize(self, chunksize)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.setRowCount(self, rowCount)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; RatZarr.writeBlock(self, colName, block, startRow)
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

### class RatZarrError
      Generic exception for RatZarr

## Functions
### def mainCmd()

## Values
    CHUNKSIZE_ATTR = CHUNKSIZE
    DFLT_CHUNKSIZE = 500000
    MODIFICATION_ATTR = MODIFICATIONLOG
    boto3 = None
    colNameByType = {<class 'numpy.int32'>: 'int32col', <class 'numpy.float32'>: 'float32col', StringDType(): 'stringcol'}
