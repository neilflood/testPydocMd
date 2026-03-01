# rios.inputcollection
    All code in this module is now deprecated (version 2.0.0)
    
    This module contains the InputCollection and InputIterator
    classes. These classes are for ImageReader to keep track
    of the inputs it has and deal with resampling.

## Classes
### class InputCollection
      InputCollection class. Keeps all of the inputs and the
      information we need about them in one place.
      
      Gets passed a list of filenames and opens them, creates
      a PixelGridDefn and pulls out their null values.
      
      Setting the reference dataset is done via setReference()
      (is first dataset by default). Resampling can be 
      performed by resampleToReference(). 
      
      Use checkAllMatch() to see if resampling is necessary.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.\_\_init\_\_(self, imageList, loggingstream=<_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>)
          Constructor. imageList is a list of file names that 
          need to be opened. An ImageOpenException() is raised
          should opening fail.
          
          Set loggingstream to a file like object if you wish
          logging about resampling to be sent somewhere else
          rather than stdout.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.checkAllMatch(self)
          Returns whether any resampling necessary to match
          reference dataset.
          
          Use as a check if no resampling is done that we
          can proceed ok.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.cleanup(self)
          Removes any temp files. To be called from destructor?
          Seems not given Neil's experience - needs to be called
          from finally clause - not sure how to enforce this
          in user's script....

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.close(self)
          Closes all open datasets

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.findWorkingRegion(self, footprint)
          Work out what the combined pixel grid should be, in terms of the 
          given reference input raster. Returns a PixelGridDefn object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.makePixGridFromDataset(ds)
          Make a pixelgrid object from the given dataset. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.makeWarpNullOptions(nullValList)
          Make appropriate options for gdalwarp, to handle 
          null value properly. If any of the null values
          in the list is None, then we can't do it because the null 
          value is not set on the file, so the return value is None. 
          
          Returns a list that can be passed to as srcNodata and
          dstNodata params to gdal.Warp().

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.resampleAllToReference(self, footprint, resamplemethodlist, tempdir='.', useVRT=False, allowOverviewsGdalwarp=False)
          Reamples all datasets that don't match the reference to the
          same as the reference.
          
          footprint is imageio.INTERSECTION, imageio.UNION or imageio.BOUNDS_FROM_REFERENCE
          resamplemethod is a string containing a method supported by gdalwarp.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.resampleToReference(self, ds, nullValList, workingRegion, resamplemethod, tempdir='.', useVRT=False, allowOverviewsGdalwarp=False)
          Resamples any inputs that do not match the reference, to the 
          reference image. 
          
          ds is the GDAL dataset that needs to be resampled.
          nullValList is the list of null values for that dataset.
          workingRegion is a PixelGridDefn
          resamplemethod is a string containing a method supported by gdalwarp.
          
          Returns the new (temporary) resampled dataset instance.
          
          Do not call directly, use resampleAllToReference()

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.setReference(self, refpath=None, refgeotrans=None, refproj=None, refNCols=None, refNRows=None, refPixgrid=None)
          Sets the reference dataset for resampling purposes.
          
          Either set refpath to a path to a GDAL file and all
          necessary values will be extracted. Or pass refgeotrans,
          refproj, refNCols and refNRows.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputCollection.specialProjFixes(projwkt)
          Does any special fixes required for the projection. Returns the fixed
          projection WKT string. 
          
          Specifically this does two things, both of which are to cope with rubbish
          that Imagine has put into the projection. Firstly, it removes the
          crappy TOWGS84 parameters which Imagine uses for GDA94, and secondly 
          removes the crappy name which Imagine gives to the correct GDA94.
          
          If neither of these things is found, returns the string unchanged. 

### class InputIterator
      Class to allow iteration across an InputCollection instance.
      Do not instantiate this class directly - it is created
      by InputCollection.__iter__().
      
      See http://docs.python.org/library/stdtypes.html#typeiter
      for a description of how this works. There is another way,
      see: http://docs.python.org/reference/expressions.html#yieldexpr
      but it seemed too much like Windows 3.1 programming which
      scared me!
      
      Returns a image name, GDAL Dataset, PixelGridDefn, nullvallist and datatype
      for each iteration

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputIterator.\_\_init\_\_(self, collection)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InputIterator.next(self)

