# rios.applier
    Basic tools for setting up a function to be applied over 
    a raster processing chain. The :func:`rios.applier.apply` function is the main
    point of entry in this module. 

## Classes
### class ApplierControls
      Controls for the operation of rios, for use with 
      the :func:`rios.applier.apply` function. 
      
      This object starts with default values for all controls, and 
      has methods for setting each of them to something else. 
      
      Default values are provided for all attributes, and can then be over-ridden
      with the 'set' methods given.
      
      Some 'set' methods take an optional imagename argument. If given, this shouldbe 
      the same internal name used for a given image as in the :class:`rios.structures.FilenameAssociations`
      objects. This is the internal name for that image, and the method will set
      the parameter in question for that specific image, which will over-ride the
      global value set when no imagename is given.
      
      Attributes are:
          * **windowxsize**     X size of rios block (pixels)
          * **windowysize**     Y size of rios block (pixels)
          * **overlap**         Number of pixels in margin for block overlaps
          * **footprint**       :data:`rios.applier.INTERSECTION` or :data:`rios.applier.UNION` or :data:`rios.applier.BOUNDS_FROM_REFERENCE`
          * **drivername**      GDAL driver short name for output
          * **creationoptions** GDAL creation options for output
          * **thematic**        True/False for thematic outputs
          * **layernames**      List of layer names for outputs
          * **referenceImage**  Image for reference projection and grid that inputs will be resampled to.
          * **referencePixgrid** pixelGrid for reference projection and grid
          * **loggingstream**   file-like for logging of messages
          * **progress**        progress object
          * **statsIgnore**     global stats ignore value for output (i.e. null value)
          * **inputnodata**     Over-ride of null value for input file, in reprojecting
          * **calcStats**       Obsolete. See setCalcStats() docstring
          * **omitPyramids**    Boolean to omit pyramid layers in outputs
          * **omitBasicStats**  Boolean to omit basic statistics in outputs
          * **omitHistogram**   Boolean to omit histogram in outputs
          * **overviewLevels**  List of level factors used when calculating output image overviews
          * **overviewMinDim**  Minimum dimension of highest overview level
          * **overviewAggType** Aggregation type for calculating overviews
          * **singlePassPyramids**    Boolean to do pyramids on outputs in a single pass
          * **singlePassBasicStats**  Boolean to do basic stats on outputs in a single pass
          * **singlePassHistogram**   Boolean to do histogram on outputs in a single pass
          * **tempdir**         Name of directory for temp files (resampling, etc.)
          * **resampleMethod**  String for resample method, when required (as per GDAL)
          * **numThreads**      Deprecated. Number of parallel threads used for processing each image block
          * **jobManagerType**  Deprecated. Which :class:`rios.parallel.jobmanager.JobManager` sub-class to use for parallel processing (by name)
          * **concurrency**     Instance of :class:`rios.structures.ConcurrencyStyle` (use instead of numThreads/jobManagerType)
          * **autoColorTableType** Type of color table to be automatically added to thematic output rasters
          * **allowOverviewsGdalwarp** Allow use of overviews in input resample (dangerous, do not use)
          * **approxStats**       Allow approx stats (much faster)
          * **layerselection**  List of selected layer numbers for input
          * **jobName**         String name for this job, for cosmetic use only
          * **callBeforeClose**   Function and args to call before closing output file
      
      Options relating to vector input files
          * **burnvalue**       Value to burn into raster from vector
          * **filtersql**       SQL where clause used to filter vector features
          * **alltouched**      Boolean. If True, all pixels touched are included in vector. 
          * **burnattribute**   Name of vector attribute used to supply burnvalue
          * **vectorlayer**     Number (or name) of vector layer
          * **vectordatatype**  Numpy datatype to use for raster created from vector
          * **vectornull**      Rasterised vector is initialised to this value, before burning

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls._checks(self, infiles, outfiles)
        Run a few simple checks on settings on the controls object.
        Raise an exception if an error is found.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.emulateOldJobManager(self)
        Uses the new ConcurrencyStyle model (version 2.0.0) to emulate the
        old JobManager concurrency. The new stuff is much better, but this
        allows old programs to use it without modification. Prints a
        deprecation warning. This routine is called automatically if the
        old JobManager settings have been invoked, and should not be used
        otherwise.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.getOptionForImagename(self, option, imagename)
        Returns the value of a particular option for the 
        given imagename. If only the global option has been set,
        then that is returned, but if a specific value has been set for 
        the given imagename, then use that. 
        
        The imagename is the same internal name as used for the image
        in the :class:`rios.structures.FilenameAssociations` objects. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.selectInputImageLayers(self, layerselection, imagename=None)
        Set which layers are to be read from the input image(s). Default
        will read all layers. If imagename is given, selection will be for 
        that image only. The layerselection parameter should be a list
        of layer numbers. Layer numbers follow GDAL conventions, i.e. 
        a layer number of 1 refers to the first layer in the file. 
        Can  be much more efficient when only using a small subset of 
        layers from the inputs.
        
        New in version 1.4.0

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setAllowOverviewsGdalwarp(self, allowOverviewsGdalwarp)
        This option is provided purely for testing purposes, and it is recommended 
        that this never be used operationally. 
        
        In GDAL >= 2.0, the default behaviour of gdalwarp was modified so that it
        will use overviews during a resample to a lower resolution. By default, 
        RIOS now switches this off again (by giving gdalwarp the '-ovr NONE' 
        switch), as this is very unreliable behaviour. Overviews can be 
        calculated by many different methods, and the user of the 
        file cannot tell how they were done. 
        
        In order to allow users to assess the damage done by this, we provide
        this option to allow resampling to use overviews. This also allows
        compatibility with versions of RIOS which did not switch it off, before 
        we discovered that it was happening. To allow this, set this parameter
        to True, otherwise it defaults to False. 
        
        We strongly recommend against allowing gdalwarp to use overviews. 
        
        New in version 1.4.8

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setAlltouched(self, alltouched, vectorname=None)
        Set boolean value of alltouched attribute. If alltouched is True, then
        pixels will count as "inside" a vector polygon if they touch the polygon,
        rather than only if their centre is inside. 
        If vectorname given, then set only for that vector.
        
        Default is False.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setApproxStats(self, approxStats, imagename=None)
        Set boolean value of approxStats attribute. This modifies the
        computation of both basic statistics and histograms on output
        files, allowing the use of lower resolution pyramid layers
        (i.e. overviews), instead of the full resolution data. This
        dramatically speeds up both calculations.
        
        This has no effect when using single-pass calculation for basic
        statistics and histograms (new in 2.0.5), and so setting this to
        be True will have the effect of disabling single-pass for both
        basic statistics and histograms. It is independent of single-pass
        pyramids layers.
        
        If imagename is given, then the setting will apply only to
        the given image.
        
        New in version 1.4.9

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setAutoColorTableType(self, autoColorTableType, imagename=None)
        If this option is set, then thematic raster outputs will have a
        color table automatically generated and attached to them. The type is
        passed to :func:`rios.rat.genColorTable` to determine what type of automatic
        color table is generated. 
        
        The default type will be taken from $RIOS_DFLT_AUTOCOLORTABLETYPE if it
        is set. If that is not set, then the default is not to automatically attached
        any color table to thematic output rasters.
        
        In practise, it is probably simpler to explicitly set the color table using 
        the :func:`rios.rat.setColorTable` function, after creating the file, but this
        is an alternative. 
        
        Note that the imagename parameter can be given, in which case the autoColorTableType 
        will only be applied to that raster. 
        
        None of this has any impact on athematic outputs. 
        
        New in version 1.4.3

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setBurnAttribute(self, burnattribute, vectorname=None)
        Set the vector attribute name from which to get the burn value
        for each vector feature. If vectorname is given, set only for that
        vector input. Default is to use burnvalue instead of burnattribute. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setBurnValue(self, burnvalue, vectorname=None)
        Set the burn value to be used when rasterizing the input vector(s).
        If vectorname given, set only for that vector. Default is 1. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setCalcStats(self, calcStats, imagename=None)
        From version 2.0.5, this is now obsolete.
        
        The default behaviour is to calculate pyramid layers, basic statistics,
        and histogram on all outputs. To omit any of these, call the
        associated controls method (setOmitPyramids, setOmitBasicStats,
        or setOmitHistogram).
        
        In earlier versions, these were all controlled with a call to this
        method. It is now preferred that these omit methods be called
        individually. The setCalcStats method now emulates the old behaviour,
        but will probably be deprecated in some future version, and
        eventually removed.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setCallBeforeClose(self, func, args=(), imagename=None)
        Set a function and its arguments, to be called on an output image,
        just before it is closed.
        
        args is a tuple of arguments to the function (default is an empty tuple).
        The function will be called with the still-open GDAL Dataset object as
        its first argument, and then all the arguments in args, as follows::
        
            func(dataset, *args)

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setConcurrencyStyle(self, concurrencyStyle)
        Set the concurrency style. Argument is an instance of the
        :class:`rios.structures.ConcurrencyStyle` class. See there
        for full details of how to use this.
        
        New in version 2.0

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setCreationOptions(self, creationoptions, imagename=None)
        Set a list of GDAL creation options (should match with the driver). 
        Each list element is a string of the form "NAME=VALUE". 
        
        Defaults are suitable for the default driver, and need to be changed
        if that is changed. However, if an appropriate driver-specific default 
        environment variable ($RIOS_DFLT_CREOPT_<driver>) is given, this 
        will be used. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setFilterSQL(self, filtersql, vectorname=None)
        Set an SQL WHERE clause which will be used to filter vector features.
        If vectorname is given, then set only for that vector

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setFootprintType(self, footprint)
        Set type of footprint, one of INTERSECTION, UNION or 
        BOUNDS_FROM_REFERENCE from this module
        
        The footprint type controls the extent of the pixel grid
        used for calculation within the user function, and of the
        output files.
        
        Using INTERSECTION will result in the maximum extent which
        is wholly included in all of the input images. Using UNION results
        in the minimum extent which wholly includes all of the input
        images. If BOUNDS_FROM_REFERENCE is used, then the extent will
        be the same as that of the reference image or pixgrid, regardless
        of the extents of the various other inputs.
        
        For both UNION and BOUNDS_FROM_REFERENCE, it is possible to
        have pixels which are within the extent, but outside one or
        more of the input files. The input data for such pixels are filled
        with the null value for that file. If no null value is set for that
        file, then zero is used.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setInputNoDataValue(self, nodataValue, imagename=None)
        Set a 'no data' value for input file(s). This over-rides whatever
        null value may be defined within the file itself, and most importantly,
        will over-ride when the file does not have a null value set at all.
        
        The main reason this is important is when an input file does not
        have a null value, and is being reprojected on input. Since it has no
        null value, the resampling on the edge of any null value region in the
        image will risk blurring the nulls into the non-null area. Normally
        this is avoided if the file has a null value set. If the input file
        is not under the control of the user, then it cannot be set before
        reading, so this allows the user to over-ride it, and behave as though
        it had been set on the original file.
        
        If the supplied value is a single scalar, it will apply to all bands
        of the input, but if it is a list of values, they will apply
        per-band.
        
        If there is no reprojection on input, then this setting will have no
        effect on the data. However, it will be honoured by a call to
        info.getNoDataValueFor(), inside the user function (although that
        is itself discouraged).
        
        If the ``imagename`` parameter is used, then the setting will apply
        only to that input, otherwise it will be applied to all inputs.
        
        New in version 2.0.0

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setJobManagerType(self, jobMgrType)
        This is now deprecated (version 2.0.0).
        Please see setConcurrencyStyle instead.
        
        Set which type of JobManager is to be used for parallel processing.
        See :mod:`rios.parallel.jobmanager` for details. Default is taken from
        $RIOS_DFLT_JOBMGRTYPE. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setJobName(self, jobName)
        Set a job name string. This is entirely optional, and has only cosmetic
        effect.
        
        A job name string is set on the controls object, and is made available
        to things like other compute workers. This can assist in identifying
        workers and relating them to the originating job, in situations where
        there are multiple main jobs running simultaneously.
        
        It defaults to None, and is then unused.
        
        New in version 2.0.5

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setLayerNames(self, layerNames, imagename=None)
        Set list of layernames to be given to the output file(s). This is not
        really well supported by most format drivers, and should probably
        be avoided. It seemed like a good idea at the time.
        
        New in version 1.1.5

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setLoggingStream(self, loggingstream)
        Set the rios logging stream to the given file-like object.
        
        This is now deprecated (v2.0.0), and has no effect. The loggingstream
        is no longer used within RIOS.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setNumThreads(self, numThreads)
        This is now deprecated (version 2.0.0).
        Please see setConcurrencyStyle instead.
        
        Set the number of 'threads' to be used when processing each block 
        of imagery. Note that these are not threads in the technical sense, 
        but are handled by the JobManager class, and are some form of 
        cooperating parallel processes, depending on the type of job 
        manager sub-class selected. See :mod:`rios.parallel.jobmanager` 
        for full details. Note that this is only worth using on very 
        computationally-intensive tasks. Default is 1, i.e. no parallel 
        processing. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOmitBasicStats(self, omitBasicStats, imagename=None)
        The default behaviour is to calculate basic statistics
        on all output images. To omit these, call setOmitBasicStats(True).
        
        If imagename is given, it should be a symbolic name as used on
        the outfiles object, and this setting will apply only to that
        image.
        
        New in version 2.0.5.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOmitHistogram(self, omitHistogram, imagename=None)
        The default behaviour is to calculate image histograms
        on all output images. To omit these, call setOmitHistogram(True).
        
        If imagename is given, it should be a symbolic name as used on
        the outfiles object, and this setting will apply only to that
        image.
        
        New in version 2.0.5.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOmitPyramids(self, omitPyramids, imagename=None)
        The default behaviour is to calculate pyramid layers (i.e. overviews)
        on all output images. To omit these, call setOmitPyramids(True). 
        
        If imagename is given, it should be a symbolic name as used on
        the outfiles object, and this setting will apply only to that
        image.
        
        New in version 1.1.2.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOptionForImagename(self, option, imagename, value)
        Set the given option specifically for the given imagename. This 
        method is for internal use only. If you wish to set a particular 
        attribute, use the corresponding 'set' method. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOutputDriverName(self, drivername, imagename=None)
        Set the output driver name to the given GDAL shortname.
        
        Note that the GDAL creation options have defaults suitable only 
        for the default driver, so if one sets the output driver, then 
        the creation options should be reviewed too. 
        
        In more recent versions of RIOS, the addition of driver-specific
        default creation options ($RIOS_DFLT_CREOPT_<driver>) allows for
        multiple default creation options to be set up.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOverlap(self, overlap)
        Set the overlap to the given value.
        
        Overlap is a number of pixels, and is somewhat mis-named. It refers
        to the amount of margin added to each block of input, so that the
        blocks will overlap, hence the actual amount of overlap is really
        double this value.
        
        The margin can result in pixels which are outside the extent of
        the given input images. These pixels will be filled with the null
        value for that input file, or zero if no null value is set on
        that file.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOverviewAggregationType(self, overviewAggType, imagename=None)
        Set the type of aggregation used when computing overview images (i.e. pyramid 
        layers). Normally a thematic image should be aggregated using "NEAREST", while a 
        continuous image should be aggregated using "AVERAGE". When the setting is 
        given as None, then a default is used. If using an output format which 
        supports LAYER_TYPE, the default is based on this, but if not, it comes from 
        the value of the environment variable $RIOS_DEFAULT_OVERVIEWAGGREGATIONTYPE.
        
        This method should usually be used to set when writing an output to a format
        which does not support LAYER_TYPE, and which is not appropriate for the
        setting given by the environment default. 
        
        New in version 1.4.1

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOverviewLevels(self, overviewLevels, imagename=None)
        Set the overview levels to be used on output images (i.e. pyramid layers). 
        Levels are specified as a list of integer factors, with the same meanings 
        as given to the gdaladdo command. 
        
        New in version 1.4.1

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setOverviewMinDim(self, overviewMinDim, imagename=None)
        Set minimum dimension allowed for output overview. Overview levels (i.e. pyramid
        layers) will be calculated as per the overviewLevels list of factors, but 
        only until the minimum dimension falls below the value of overviewMinDim
        
        New in version 1.4.1

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setProgress(self, progress)
        Set the progress display object. Default is no progress
        object.
        
        The progress object should be an instance of one of the classes
        from :class:`rios.cuiprogress`, and is used to generate a simple
        progress indicator showing the percentage completed.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setReferenceImage(self, referenceImage)
        Set the name of the image to use for the reference pixel grid and 
        projection. If neither referenceImage nor referencePixgrid are set, 
        then no resampling will be allowed. Only set one of referenceImage or
        referencePixgrid. 
        
        The reference image can be given as either the internal name, as given
        on infiles, or the external filename. The internal name is
        preferred, and consistent with other usage, but the filename is
        allowed for backward compatibility.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setReferencePixgrid(self, referencePixgrid)
        Set the reference pixel grid. If neither referenceImage nor 
        referencePixgrid are set, then no resampling will be allowed. 
        Only set one of referenceImage or referencePixgrid. The referencePixgrid
        argument is of type :class:`rios.pixelgrid.PixelGridDefn`. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setResampleMethod(self, resampleMethod, imagename=None)
        Set resample method to be used for resampling of input files. Possible
        options are those defined by gdalwarp, i.e. 'near', 'bilinear', 
        'cubic', 'cubicspline', 'lanczos'. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setSinglePassBasicStats(self, singlePassBasicStats, imagename=None)
        The default behaviour is to attempt to compute basic statistics
        (i.e. min/max/mean/stddev) for all output files as each block is
        computed. This avoids an extra pass through the data afterwards.
        
        If singlePassBasicStats is given here as False, then this will not be
        attempted, and instead GDAL's ComputeStatistics() function will be
        called after the output is completed (i.e. a whole extra pass through
        the data).
        
        New in version 2.0.5.
        
        See also setApproxStats() for an alternative way of speeding up
        basic statistics.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setSinglePassHistogram(self, singlePassHistogram, imagename=None)
        The default behaviour is to attempt to compute a histogram
        (on each band) for all output files. If possible, this will be done
        incrementally, as each block of output is written, but if that is
        not possible, it will be done at the end of processing, which will
        require an extra pass through each output file.
        
        The single-pass histogram is only supported for integer datatypes. The
        default behaviour is to do single-pass histograms if possible, and
        if not, to fall back computing histograms using GDAL's GetHistogram()
        function, after the output files have been written.
        
        If singlePassHistogram is given here as False, then GDAL's function
        will always be used. If singlePassHistogram is given here as True,
        and the required conditions are not satisfied, then an exception
        will be raised, explaining why this cannot be done.
        
        New in version 2.0.5.
        
        See also setApproxStats() for an alternative way of speeding up
        histogram calculation.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setSinglePassPyramids(self, singlePassPyramids, imagename=None)
        The default behaviour is to attempt to compute pyramid layers (i.e.
        overviews) for output files as each block is computed. This avoids
        an extra pass through the data afterwards.
        
        The single-pass pyramids requires that incremental writing of
        pyramids is supported by the output format driver. This is checked
        for each driver used. The default will use it if supported, but fall
        back to GDAL if not.
        
        If singlePassPyramids is given here as False, then this will not be
        attempted, and instead GDAL's BuildOverviews() function will be called
        after the output is completed (i.e. a whole extra pass through the
        data). If True is given, and the driver does not support it, then
        an exception will be raised.
        
        New in version 2.0.5.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setStatsIgnore(self, statsIgnore, imagename=None)
        Set the 'no data' value for the output files (also known as the
        'null' value). This value will be written into the output files,
        and will thus be ignored when calculating statistics, histograms
        and overviews (pyramid layers) on those files.
        
        If this value is given as None, then no null value will be set on
        output files.
        
        If imagename is given, the setting will only apply to that image,
        otherwise it will apply to all output files.
        
        There is currently no mechanism to set different null values for
        different layers in an output file.
        
        The default value is 0. This was probably a bad idea, but to avoid
        breaking old scripts, we are not likely to change this.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setTempdir(self, tempdir)
        Set directory to use for temporary files for resampling, etc.
        
        Default is '.' (i.e. current directory).

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setThematic(self, thematicFlag, imagename=None)
        Boolean flag to indicate whether the output file is thematic. A value
        of True means the output will be set as thematic, although this may
        not be supported by the output format driver.
        
        Default is False (i.e. not thematic).

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setVectorDatatype(self, vectordatatype, vectorname=None)
        Set numpy datatype to use for rasterized vectors. If vectorname
        given, set only for that vector.
        
        Default is numpy.uint8

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setVectorNull(self, vectornull, vectorname=None)
        Set the vector null value. This is used to initialise the
        rasterised vector, before burning in the burn value. This is of most
        importance when burning values from a vector attribute column, as 
        this should be a distinct value from any of the values in the column. 
        If this is not so, then polygons can end up blending with the background,
        resulting in incorrect answers.
        
        Default is 0

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setVectorlayer(self, vectorlayer, vectorname=None)
        Set number/name of vector layer, for vector formats which have 
        multiple layers. Not required for plain shapefiles. 
        Can be either a layer number (start at zero) or 
        a layer name. If vectorname given, set only for that vector.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setWindowSize(self, windowxsize, windowysize)
        Sets the X and Y size of the blocks used in one call.
        Images are processed in blocks (windows) of 'windowxsize' 
        columns, and 'windowysize' rows.
        
        New in version 1.4.17.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setWindowXsize(self, windowxsize)
        Set the X size of the blocks used. Images are processed in 
        blocks (windows) of 'windowxsize' columns, and 'windowysize' rows. 

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierControls.setWindowYsize(self, windowysize)
        Set the Y size of the blocks used. Images are processed in 
        blocks (windows) of 'windowxsize' columns, and 'windowysize' rows. 

### class ApplierProgress
      Wrapper around the controls progress object, just to simplify the
      update call, and keeping track of whether the percentage has changed.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierProgress.\_\_init\_\_(self, controls, numBlocks)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierProgress.update(self, blockNdx)

## Functions
### def apply(userFunction, infiles, outfiles, otherArgs=None, controls=None)
        Apply the given 'userFunction' to the given
        input and output files. 
        
        infiles and outfiles are :class:`rios.structures.FilenameAssociations` objects to 
        define associations between internal variable names and
        external filenames, for the raster file inputs and outputs. 
        
        otherArgs is an object of extra arguments to be passed to the 
        userFunction, each with a sensible name on the object. These 
        can be either input or output arguments, entirely at the discretion
        of userFunction(). otherArgs should be in instance of :class:`rios.structures.OtherInputs`
        
        The userFunction has the following call sequence::
        
            userFunction(info, inputs, outputs)
        
        or::
        
            userFunction(info, inputs, outputs, otherArgs)
        
        if otherArgs is not None.
        
        inputs and outputs are objects in which there are named attributes 
        with the same names as those given in the infiles and outfiles 
        objects. In the inputs and outputs objects, available inside 
        userFunction, these attributes contain numpy arrays of data read 
        from/written to the corresponding image file. 
        
        If the attributes given in the infiles or outfiles objects are 
        lists of filenames, the the corresponding attributes of the 
        inputs and outputs objects inside the applied function will be 
        lists of image data blocks instead of single blocks. 
        
        The numpy arrays are always 3-d arrays, with shape::
        
            (numBands, numRows, numCols)
        
        The datatype of the output image(s) is determined directly
        from the datatype of the numpy arrays in the outputs object. 
        
        The info object contains many useful details about the processing, 
        and will always be passed to the userFunction. It can, of course, 
        be ignored. It is an instance of the :class:`rios.readerinfo.ReaderInfo` class. 
        
        The controls argument, if given, is an instance of the 
        :class:`rios.applier.ApplierControls` class, which allows control of various 
        aspects of the reading and writing of images. See the class 
        documentation for further details.
        
        The apply function returns a :class:`rios.structures.ApplierReturn` object
        (new in version 2.0).
        
        There is a page dedicated to :doc:`applierexamples`.

### def apply_multipleCompute(userFunction, infiles, outfiles, otherArgs, controls, allInfo, workinggrid, blockList)
        Called internally from the apply() function. Not to be called directly.
        
        Apply function for the multiple compute cases. Starts a number of
        compute workers, each of which calls the user function on inputs
        and creates outputs, which this function writes to the output files.

### def apply_singleCompute(userFunction, infiles, outfiles, otherArgs, controls, allInfo, workinggrid, blockList, outBlockBuffer, inBlockBuffer, workerID, forceExit)
        Called internally from the apply() function. Not to be called directly.
        
        Apply function for simplest configuration, with no compute concurrency.
        Does have possible read concurrency.
        
        This function is also called for each compute worker in the
        batch-oriented compute worker styles, where each worker is an instance
        of a single-compute case.

### def checkAllMatch(pixgridList, refPixGrid)
        Returns whether any resampling necessary to match
        reference dataset.
        
        Use as a check if no resampling is done that we
        can proceed ok.

### def makeBlockList(workinggrid, controls)
        Divide the working grid area into blocks. Return a list of
        ApplierBlockDefn objects

### def makeWorkingGrid(infiles, allInfo, controls)
        Work out the projection and extent of the working grid.
        
        Return a PixelGridDefn object representing it.

### def readAllImgInfo(infiles)
        Open all input files and create an ImageInfo (or VectorFileInfo)
        object for each. Return a dictionary of them, keyed by their
        position within infiles, i.e. (symbolicName, SeqNum).

### def reportWorkerException(exceptionRecord)
        Report the given WorkerExceptionRecord object to stderr

## Values
    BOUNDS_FROM_REFERENCE = 2
    CW_AWSBATCH = 'CW_AWSBATCH'
    CW_ECS = 'CW_ECS'
    CW_NONE = 'CW_NONE'
    CW_PBS = 'CW_PBS'
    CW_SLURM = 'CW_SLURM'
    CW_SUBPROC = 'CW_SUBPROC'
    CW_THREADS = 'CW_THREADS'
    DEFAULTCREATIONOPTIONS = ['COMPRESSED=TRUE', 'IGNOREUTM=TRUE']
    DEFAULTDRIVERNAME = 'HFA'
    DEFAULTFOOTPRINT = 0
    DEFAULTLOGGINGSTREAM = <_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
    DEFAULTOVERLAP = 0
    DEFAULTWINDOWXSIZE = 256
    DEFAULTWINDOWYSIZE = 256
    DEFAULT_AUTOCOLORTABLETYPE = None
    DEFAULT_MINOVERVIEWDIM = 128
    DEFAULT_OVERVIEWAGGREGRATIONTYPE = 'NEAREST'
    DEFAULT_OVERVIEWLEVELS = [4, 8, 16, 32, 64, 128, 256, 512]
    DEFAULT_RESAMPLEMETHOD = 'near'
    INTERSECTION = 0
    UNION = 1
    dfltDriverOptions = {'HFA': ['COMPRESSED=TRUE', 'IGNOREUTM=TRUE'], 'GTiff': ['TILED=YES', 'COMPRESS=LZW', 'INTERLEAVE=BAND', 'BIGTIFF=IF_SAFER']}
