# rios.imageio
    The only things of value left in this module are the original definitions of
    UNION, INTERSECTION and BOUNDS_FROM_REFERENCE.
    
    There are also two functions wld2pix and pix2wld, and the Coord class they use.
    These should also be deprecated, in favour of GDAL's ApplyGeoTransform
    and InvGeoTransform (which they now use internally anyway).
    However, they are used in public-facing ways in the ReaderInfo object,
    so removing them would, in principle, be a breaking change.
    They are harmless enough, so they have been left here.
    
    In general, this module should be ignored.

## Classes
### class Coord
      a simple class that contains one coord

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Coord.\_\_init\_\_(self, x, y)
          Initialize self.  See help(type(self)) for accurate signature.

## Functions
### def GDALTypeToNumpyType(gdaltype)
        This function is deprecated.
        Use gdal_array.GDALTypeCodeToNumericTypeCode instead.
        
        Given a gdal data type returns the matching numpy data type

### def NumpyTypeToGDALType(numpytype)
        This function is deprecated.
        Use gdal_array.NumericTypeCodeToGDALTypeCode instead.
        
        For a given numpy data type returns the matching GDAL data type

### def pix2wld(transform, x, y)
        converts a set of pixels coords to map coords

### def wld2pix(transform, geox, geoy)
        converts a set of map coords to pixel coords

## Values
    BOUNDS_FROM_REFERENCE = 2
    INTERSECTION = 0
    UNION = 1
