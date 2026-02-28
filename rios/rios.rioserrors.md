# rios.rioserrors
    All exceptions used within rios. 

## Classes
### class ArrayShapeError
      Error in shape of an array

### class AttributeTableColumnError
      Unable to find specified column

### class AttributeTableTypeError
      Type does not match that expected

### class ColorTableGenerationError
      Error generating a color table

### class ECSError
      Error arising from AWS ECS

### class FileOpenError
      Failed to open an input or output file

### class GDALLayerNumberError
      A GDAL layer number was given, but was out of range

### class GdalWarpError
      Error while running gdalwarp

### class ImageOpenError
      Image wasn't able to be opened by GDAL

### class IntersectionError
      Images don't have a common area

### class JobMgrError
      Errors from Jobmanager class

### class KeysMismatch
      Keys do not match expected

### class MismatchedListLengthsError
      Two lists had different lengths, when they were supposed to be the same length

### class OutsideImageBoundsError
      Requested Block is not available

### class ParameterError
      Incorrect parameters passed to function

### class PermissionError
      Error due to permissions on temp files

### class ProcessCancelledError
      Process was cancelled by user

### class RatBlockLengthError
      Error with RAT block length, in ratapplier

### class RatMismatchError
      Inconsistent RATs on inputs to ratapplier

### class ResampleNeededError
      Images do not match - resample needs to be turned on

### class RiosError
      Base class for RIOS exceptions

### class SinglePassActionsError
      An error in processing single-pass actions

### class ThematicError
      File unable to be set to thematic

### class TimeoutError
      Something timed out

### class TypeConversionError
      Unknown type conversion

### class UnavailableError
      A dependency is unavailable

### class VectorAttributeError
      Unable to find specified index in vector file

### class VectorGeometryTypeError
      Unexpected Geometry type

### class VectorLayerError
      Unable to find the specified layer

### class VectorProjectionError
      Vector projection does not match raster projection

### class VectorRasterizationError
      Rasterisation of Vector dataset failed

### class WorkerExceptionError
      A worker thread or process has raised an exception

### class WrongControlsObject
      The wrong type of control object has been passed to apply

## Functions
### def deprecationWarning(msg, stacklevel=2)
        Print a deprecation warning to stderr. Includes the filename
        and line number of the call to the function which called this.
        The stacklevel argument controls how many stack levels above this
        gives the line number.
        
        Implemented in mimcry of warnings.warn(), which seems very flaky.
        Sometimes it prints, and sometimes not, unless PYTHONWARNINGS is set
        (or -W is used). This function at least seems to work consistently.

## Values
    deprecationAlreadyWarned = set()
