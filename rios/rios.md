# rios
    Raster Input Output Simplification
    
    A Python package to simplify raster input/output
    using GDAL, allowing a programmer to focus on the
    processing of the data rather than the mechanics of
    raster I/O. 
    
    Rios is built on top of GDAL, and handles, among other things: 
        - Opening raster files and reading/writing raster data
        - Determining the area of intersection (or union) of 
          input rasters
        - Reprojecting inputs to be on the same projection and
          pixel alignment
        - Stepping through the rasters in small blocks, to avoid 
          large memory usage
    
    The main starting point is the rios.applier.apply function.

## Classes
### class VersionObj
      Our own replacement for the old LooseVersion class,
      since they deprecated that.
      
      An instance of this class is initialized with a SemVer-style
      version string, and instances can be compared for ordering in
      a sensible way.
      
      Yes, I know we could use setuptools.pkg_resources, or similar,
      but I am sick of relying on other people's packages, when they
      go and change them so easily. This will be constant. Sigh.....

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; VersionObj.\_\_init\_\_(self, vstring)
          vstring is a version string, something like "a.b.c"
          where a, b & c are integers

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; VersionObj.leadingDigits(s)
          Return the leading numeric digits of the given string,
          up to anything non-digit.
          
          Clever people would use a regular expression for this, but
          I am not clever.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; VersionObj.parse(vstring)
          Return a tuple of version components.
          
          Currently wants everything to be an int. Any trailing non-digits
          will be ignored. This means that things like '2beta1' will be
          treated the same as '2'.

## Values
    RIOS_VERSION = '2.0.9'
