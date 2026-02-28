# rios.vectorreader
    This module contains the Vector and VectorReader
    class that perform on-the-fly rasterization of
    vectors into raster blocks to fit in with
    ImageReader and ImageWriter classes.
    
    As of version 2.0.0, everything in this module is deprecated. Vector reading
    is now handled much more neatly within the imagereader module.

## Classes
### class Vector
      Class that holds information about a vector dataset and how it
      should be rasterized. Used for passing to VectorReader.

#### &nbsp;&nbsp;&nbsp;&nbsp; Vector.\_\_init\_\_(self, filename, inputlayer=0, burnvalue=1, attribute=None, filter=None, alltouched=False, datatype=<class 'numpy.uint8'>, tempdir='.', driver='HFA', driveroptions=['COMPRESSED=TRUE', 'IGNOREUTM=TRUE'], nullval=0)
        Constructs a Vector object. filename should be a path to an OGR readable
        dataset. 
        inputlayer should be the OGR layer number (or name) in the dataset to rasterize.
        burnvalue is the value that gets written into the raster inside a polygon.
        Alternatively, attribute may be the name of an attribute that is looked up
        for the value to write into the raster inside a polygon.
        If you want to filter the attributes in the vector, pass filter which
        is in the format of an SQL WHERE clause.
        By default, only pixels whose centre lies within the a polygon get
        rasterised. To have all pixels touched by a polygon rasterised, set
        alltouched=True. 
        datatype is the numpy type to rasterize to - byte by default.
        tempdir is the directory to create the temporary raster file.
        driver and driveroptions set the GDAL raster driver name and options
        for the temporary rasterised file.

#### &nbsp;&nbsp;&nbsp;&nbsp; Vector.cleanup(self)
        Remove temporary file(s) and close dataset

#### &nbsp;&nbsp;&nbsp;&nbsp; Vector.matchingProj(self, proj)
        Returns True if the current vector has the same projection
        as the given projection. The projectino can be given as either
        a WKT string, or an osr.SpatialReference instance. 

#### &nbsp;&nbsp;&nbsp;&nbsp; Vector.reproject(self, proj)
        Reproject the current vector to the given projection. Places the
        result in a temporary shapefile, and opens it. Returns the ogr.Layer
        object resulting. 
         
        Assumes that shapefile format will always be sufficient - I can't think 
        of any reason why not. 
        
        This reprojection becomes a part of the "state" of the current object, i.e.
        there can only be one "reprojected" copy of the current vector. This should
        be fine. 

### class VectorReader
      Class that performs rasterization of Vector objects.

#### &nbsp;&nbsp;&nbsp;&nbsp; VectorReader.\_\_init\_\_(self, vectorContainer, progress=None)
        vectorContainer is a single Vector object, or a 
        list or dictionary that contains
        the Vector objects of the files to be read.
        If a Vector object is passed, a single block is returned
        from rasterize(), if a list is passed, 
        a list of blocks is returned, if a dictionary a dictionary is
        returned for each call to rasterize() with the same keys.
        progress is an instance of a Progress class, if none 
        an instance of cuiprogress.CUIProgress is created an used

#### &nbsp;&nbsp;&nbsp;&nbsp; VectorReader.close(self)
        Closes all datasets and removes temporary files.

#### &nbsp;&nbsp;&nbsp;&nbsp; VectorReader.rasterize(self, info)
        Rasterize the container of Vector objects passed to the 
        constuctor. Returns blocks in the same form as the 
        container passed to the constructor.

#### &nbsp;&nbsp;&nbsp;&nbsp; VectorReader.rasterizeSingle(info, vector, progress)
        Static method to rasterize a single Vector for the extents
        specified in the info object. 
        
        For efficiency, it rasterizes the whole working grid, and caches 
        this, and then reads the relevant section of the gridd for the 
        current block. 
        
        Will reproject the vector into the working projection, if required. 
        
        A single numpy array is returned of rasterized data.

## Functions
### def rasterizeProgressFunc(value, string, progressObj)
        called by gdal.RasterizeLayer

## Values
    DEFAULTBURNVALUE = 1
    DEFAULTCREATIONOPTIONS = ['COMPRESSED=TRUE', 'IGNOREUTM=TRUE']
    DEFAULTDRIVERNAME = 'HFA'
