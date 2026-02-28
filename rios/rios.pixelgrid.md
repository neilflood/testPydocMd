# rios.pixelgrid
    Utility class PixelGridDefn, which defines a pixel grid,
    plus useful operations on it. 
    
    Utility function findCommonRegion() to find the union 
    or intersection region of a list GDAL datasets, in a 
    reference grid. 

## Classes
### class PixelGridDefn
      Definition of a pixel grid, including
      the size and extent, and by implication the
      resolution and alignment. 
      
      Methods are defined for relationships with
      other instances, including:
      
          * intersection()
          * union()
          * reproject()
          * alignedWith()
          * isComparable()
      
      Attributes defined on the object:
      
          * xMin
          * xMax
          * yMin
          * yMax
          * xRes
          * yRes
          * projection
          
      The bounds define the external corners of the image, i.e. the
      top-left corner of the top-left pixel, through to the bottom-right
      corner of the bottom-right pixel. This is consistent with GDAL
      conventions.
      
      The projection is given as a WKT string.
      
      The constructor takes the projection, and EITHER a complete GDAL
      geotransform tuple, with the number of rows and columns, OR a grid
      specified with all the extent limits and the pixel resolutions
      (xRes and yRes). If the geotransform is given, then the xMin, xMax, xRes
      and so on are calculated from it.

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.\_\_init\_\_(self, geotransform=None, nrows=None, ncols=None, projection=None, xMin=None, xMax=None, yMin=None, yMax=None, xRes=None, yRes=None)
        Can define in terms of a GDAL geotransform plus
        the number of rows and columns. 
        Can also be defined by giving xMin, xMax, yMin, yMax, 
        and xRes, yRes directly. 
        
        Projection is given as a WKT string. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.alignedWith(self, other)
        Returns True if self is aligned with other. This means that
        they represent the same grid, with different extents. 
        
        Alignment is checked within a small tolerance, so that exact 
        floating point matches are not required. However, notionally it
        is possible to get a match which shouldn't be. The tolerance is
        calculated as::
        
            tolerance = 0.01 * pixsize / npix
        
        and if a mis-alignment is <= tolerance, it is assumed to be zero. 
        For further details, read the source code. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.equalPixSize(self, other)
        Returns True if pixel size of self is equal to that of other. 
        Currently only checks absolute equality, probably should 
        work out a tolerance. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.equalProjection(self, other)
        Returns True if the projection of self is the same as the 
        projection of other

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.equivalentProjection(self, otherspatialref, pixtolerance)
        Similar to equalProjection but does a less accurate test
        by checking converting coordinates from projection of self
        to otherspatialref and checking they are within pixtolerance
        pixels of each other.
        The coordinates used for this are the four corners and centre
        of image.

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.getDimensions(self)
        Utility method which returns the number of rows and columns
        in the grid. Returns a tuple::
        
            (nrows, ncols)
        
        calculated from the min/max/res values. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.getNumPix(gridMax, gridMin, gridRes)
        Works out how many pixels lie between the given min and max, 
        at the given resolution. This is for internal use only. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.intersection(self, other)
        Returns a new instance which is the intersection 
        of self and other. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.isComparable(self, other)
        Checks whether self is comparable with other. Returns
        True or False. Grids are comparable if they have equal pixel
        size, and the same projection. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.makeGeoTransform(self)
        Returns a GDAL geotransform tuple from bounds and resolution

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.reproject(self, targetGrid)
        Returns a new instance which is the reprojection
        of self to be in the same projection and pixel size
        as targetGrid

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.roundAway(x)
        Simulates Python 2 round behavour
        This is what we want as it rounds away from 0.
        The decimal module seems to be the only way to do this properly

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.snapToGrid(val, valOnGrid, res)
        Returns the nearest value to val which is a whole multiple of
        res away from valOnGrid, so that val is effectively on the same
        grid as valOnGrid. This is for internal use only. 

#### &nbsp;&nbsp;&nbsp;&nbsp; PixelGridDefn.union(self, other)
        Returns a new instance which is the union of self
        with other. 

## Functions
### def findCommonRegion(gridList, refGrid, combine=0)
        Returns a PixelGridDefn for the combination of all the grids 
        in the given gridList. The output grid is in the same coordinate 
        system as the reference grid. 
        
        The combine parameter controls whether UNION, INTERSECTION 
        or BOUNDS_FROM_REFERENCE is performed. 

### def pixelGridFromFile(filename)
        Create a PixelGridDefn object for the given image file

