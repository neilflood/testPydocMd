# processinghistory.tests
    Routine tests of processing history

## Classes
### class Fulltest(TestCase)
      Run a basic test of processing history

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.\_\_init\_\_(self, methodName='runTest')
          Create an instance of the class that will use the named test
          method when executed. Raises a ValueError if the instance does
          not have a method with the specified name.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.deleteTempFiles(filelist)
          Delete all files in the filelist

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_ancestry(self)
          Test a full ancestry tree with multiple ancestors

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_parentNoHistory(self)
          The case of a parent which has no history

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_singleFile(self)
          Test writing and reading history on a single file, for multiple drivers

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_useDataset(self)
          Test writing and reading history using an open gdal Dataset
          object instead of a filename.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Fulltest.test_vrtsupport(self)
          Test VRT support

## Functions
### def mainCmd()

### def makeRaster(filename, drvr='KEA', returnDS=False)
        Create a small raster file to use for tests.

## Values
    CHECK_AUTO_FIELDS = ['timestamp', 'login', 'cwd', 'script', 'script_dir', 'commandline', 'python_version', 'package_version_dict']
    driverList = [('KEA', 'kea'), ('HFA', 'img'), ('GTiff', 'tif')]
