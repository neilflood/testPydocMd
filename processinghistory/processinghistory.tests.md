# processinghistory.tests
    Routine tests of processing history

## Classes
### class Fulltest
      Run a basic test of processing history

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.\_\_init\_\_(self, methodName='runTest')
        Create an instance of the class that will use the named test
        method when executed. Raises a ValueError if the instance does
        not have a method with the specified name.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._addExpectedFailure(self, result, exc_info)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._addUnexpectedSuccess(self, result)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._baseAssertEqual(self, first, second, msg=None)
        The default assertEqual implementation, not type specific.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._callCleanup(self, function, /, *args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._callSetUp(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._callTearDown(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._callTestMethod(self, method)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._deprecate(original_func)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._formatMessage(self, msg, standardMsg)
        Honour the longMessage attribute when generating failure messages.
        If longMessage is False this means:
        * Use only an explicit message if it is provided
        * Otherwise use the standard message for the assert
        
        If longMessage is True:
        * Use the standard message
        * If an explicit message is provided, plus ' : ' and the explicit message

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._getAssertEqualityFunc(self, first, second)
        Get a detailed comparison function for the types of the two args.
        
        Returns: A callable accepting (first, second, msg=None) that will
        raise a failure exception if first != second with a useful human
        readable error message for those types.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest._truncateMessage(self, message, diff)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.addClassCleanup(function, /, *args, **kwargs)
        Same as addCleanup, except the cleanup items are called even if
        setUpClass fails (unlike tearDownClass).

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.addCleanup(self, function, /, *args, **kwargs)
        Add a function, with arguments, to be called when the test is
        completed. Functions added are called on a LIFO basis and are
        called after tearDown on test failure or success.
        
        Cleanup items are called even if setUp fails (unlike tearDown).

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.addTypeEqualityFunc(self, typeobj, function)
        Add a type specific assertEqual style function to compare a type.
        
        This method is for use by TestCase subclasses that need to register
        their own type equality functions to provide nicer error messages.
        
        Args:
            typeobj: The data type to call this function on when both values
                    are of the same type in assertEqual().
            function: The callable taking two arguments and an optional
                    msg= argument that raises self.failureException with a
                    useful error message when the two arguments are not equal.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertAlmostEqual(self, first, second, places=None, msg=None, delta=None)
        Fail if the two objects are unequal as determined by their
        difference rounded to the given number of decimal places
        (default 7) and comparing to zero, or by comparing that the
        difference between the two objects is more than the given
        delta.
        
        Note that decimal places (from zero) are usually not the same
        as significant digits (measured from the most significant digit).
        
        If the two objects compare equal then they will automatically
        compare almost equal.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertCountEqual(self, first, second, msg=None)
        Asserts that two iterables have the same elements, the same number of
        times, without regard to order.
        
            self.assertEqual(Counter(list(first)),
                             Counter(list(second)))
        
         Example:
            - [0, 1, 1] and [1, 0, 1] compare equal.
            - [0, 0, 1] and [0, 1] compare unequal.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertDictContainsSubset(self, subset, dictionary, msg=None)
        Checks whether dictionary is a superset of subset.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertDictEqual(self, d1, d2, msg=None)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertEqual(self, first, second, msg=None)
        Fail if the two objects are unequal as determined by the '=='
        operator.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertFalse(self, expr, msg=None)
        Check that the expression is false.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertGreater(self, a, b, msg=None)
        Just like self.assertTrue(a > b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertGreaterEqual(self, a, b, msg=None)
        Just like self.assertTrue(a >= b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertIn(self, member, container, msg=None)
        Just like self.assertTrue(a in b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertIs(self, expr1, expr2, msg=None)
        Just like self.assertTrue(a is b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertIsInstance(self, obj, cls, msg=None)
        Same as self.assertTrue(isinstance(obj, cls)), with a nicer
        default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertIsNone(self, obj, msg=None)
        Same as self.assertTrue(obj is None), with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertIsNot(self, expr1, expr2, msg=None)
        Just like self.assertTrue(a is not b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertIsNotNone(self, obj, msg=None)
        Included for symmetry with assertIsNone.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertLess(self, a, b, msg=None)
        Just like self.assertTrue(a < b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertLessEqual(self, a, b, msg=None)
        Just like self.assertTrue(a <= b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertListEqual(self, list1, list2, msg=None)
        A list-specific equality assertion.
        
        Args:
            list1: The first list to compare.
            list2: The second list to compare.
            msg: Optional message to use on failure instead of a list of
                    differences.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertLogs(self, logger=None, level=None)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertMultiLineEqual(self, first, second, msg=None)
        Assert that two multi-line strings are equal.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertNoLogs(self, logger=None, level=None)
        Fail unless no log messages of level *level* or higher are emitted
        on *logger_name* or its children.
        
        This method must be used as a context manager.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertNotAlmostEqual(self, first, second, places=None, msg=None, delta=None)
        Fail if the two objects are equal as determined by their
        difference rounded to the given number of decimal places
        (default 7) and comparing to zero, or by comparing that the
        difference between the two objects is less than the given delta.
        
        Note that decimal places (from zero) are usually not the same
        as significant digits (measured from the most significant digit).
        
        Objects that are equal automatically fail.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertNotEqual(self, first, second, msg=None)
        Fail if the two objects are equal as determined by the '!='
        operator.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertNotIn(self, member, container, msg=None)
        Just like self.assertTrue(a not in b), but with a nicer default message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertNotIsInstance(self, obj, cls, msg=None)
        Included for symmetry with assertIsInstance.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertNotRegex(self, text, unexpected_regex, msg=None)
        Fail the test if the text matches the regular expression.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertRaises(self, expected_exception, *args, **kwargs)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertRaisesRegex(self, expected_exception, expected_regex, *args, **kwargs)
        Asserts that the message in a raised exception matches a regex.
        
        Args:
            expected_exception: Exception class expected to be raised.
            expected_regex: Regex (re.Pattern object or string) expected
                    to be found in error message.
            args: Function to be called and extra positional args.
            kwargs: Extra kwargs.
            msg: Optional message used in case of failure. Can only be used
                    when assertRaisesRegex is used as a context manager.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertRegex(self, text, expected_regex, msg=None)
        Fail the test unless the text matches the regular expression.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertSequenceEqual(self, seq1, seq2, msg=None, seq_type=None)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertSetEqual(self, set1, set2, msg=None)
        A set-specific equality assertion.
        
        Args:
            set1: The first set to compare.
            set2: The second set to compare.
            msg: Optional message to use on failure instead of a list of
                    differences.
        
        assertSetEqual uses ducktyping to support different types of sets, and
        is optimized for sets specifically (parameters must support a
        difference method).

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertTrue(self, expr, msg=None)
        Check that the expression is true.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertTupleEqual(self, tuple1, tuple2, msg=None)
        A tuple-specific equality assertion.
        
        Args:
            tuple1: The first tuple to compare.
            tuple2: The second tuple to compare.
            msg: Optional message to use on failure instead of a list of
                    differences.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertWarns(self, expected_warning, *args, **kwargs)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.assertWarnsRegex(self, expected_warning, expected_regex, *args, **kwargs)
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

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.countTestCases(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.debug(self)
        Run the test without collecting errors in a TestResult

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.defaultTestResult(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deleteTempFiles(filelist)
        Delete all files in the filelist

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.doClassCleanups()
        Execute all class cleanup functions. Normally called for you after
        tearDownClass.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.doCleanups(self)
        Execute all cleanup functions. Normally called for you after
        tearDown.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.enterClassContext(cm)
        Same as enterContext, but class-wide.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.enterContext(self, cm)
        Enters the supplied context manager.
        
        If successful, also adds its __exit__ method as a cleanup
        function and returns the result of the __enter__ method.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.fail(self, msg=None)
        Fail immediately, with the given message.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deprecated_func(*args, **kwargs)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.id(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.run(self, result=None)

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.setUp(self)
        Hook method for setting up the test fixture before exercising it.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.setUpClass()
        Hook method for setting up class fixture before running tests in the class.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.shortDescription(self)
        Returns a one-line description of the test, or None if no
        description has been provided.
        
        The default implementation of this method returns the first line of
        the specified test method's docstring.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.skipTest(self, reason)
        Skip this test.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.subTest(self, msg=<object object at 0x7a8bdd7229f0>, **params)
        Return a context manager that will return the enclosed block
        of code in a subtest identified by the optional message and
        keyword parameters.  A failure in the subtest marks the test
        case as failed but resumes execution at the end of the enclosed
        block, allowing further test code to be executed.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.tearDown(self)
        Hook method for deconstructing the test fixture after testing it.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.tearDownClass()
        Hook method for deconstructing the class fixture after running all tests in the class.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_ancestry(self)
        Test a full ancestry tree with multiple ancestors

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_parentNoHistory(self)
        The case of a parent which has no history

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_singleFile(self)
        Test writing and reading history on a single file, for multiple drivers

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_useDataset(self)
        Test writing and reading history using an open gdal Dataset
        object instead of a filename.

#### &nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_vrtsupport(self)
        Test VRT support

## Functions
### def mainCmd()

### def makeRaster(filename, drvr='KEA', returnDS=False)
        Create a small raster file to use for tests.

## Values
    CHECK_AUTO_FIELDS = ['timestamp', 'login', 'cwd', 'script', 'script_dir', 'commandline', 'python_version', 'package_version_dict']
    driverList = [('KEA', 'kea'), ('HFA', 'img'), ('GTiff', 'tif')]
