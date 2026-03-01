# rios.rioserrors
    All exceptions used within rios. 

## Classes
### class ArrayShapeError(RiosError)
      Error in shape of an array

### class AttributeTableColumnError(RiosError)
      Unable to find specified column

### class AttributeTableTypeError(RiosError)
      Type does not match that expected

### class ColorTableGenerationError(RiosError)
      Error generating a color table

### class ECSError(RiosError)
      Error arising from AWS ECS

### class FileOpenError(RiosError)
      Failed to open an input or output file

### class GDALLayerNumberError(RiosError)
      A GDAL layer number was given, but was out of range

### class GdalWarpError(RiosError)
      Error while running gdalwarp

### class ImageOpenError(FileOpenError)
      Image wasn't able to be opened by GDAL

### class IntersectionError(RiosError)
      Images don't have a common area

### class JobMgrError(RiosError)
      Errors from Jobmanager class

### class KeysMismatch(RiosError)
      Keys do not match expected

### class MismatchedListLengthsError(RiosError)
      Two lists had different lengths, when they were supposed to be the same length

### class OutsideImageBoundsError(RiosError)
      Requested Block is not available

### class ParameterError(RiosError)
      Incorrect parameters passed to function

### class PermissionError(RiosError)
      Error due to permissions on temp files

### class ProcessCancelledError(RiosError)
      Process was cancelled by user

### class RatBlockLengthError(RiosError)
      Error with RAT block length, in ratapplier

### class RatMismatchError(RiosError)
      Inconsistent RATs on inputs to ratapplier

### class ResampleNeededError(RiosError)
      Images do not match - resample needs to be turned on

### class RiosError(Exception)
      Base class for RIOS exceptions

### class SinglePassActionsError(RiosError)
      An error in processing single-pass actions

### class ThematicError(RiosError)
      File unable to be set to thematic

### class TimeoutError(RiosError)
      Something timed out

### class TypeConversionError(RiosError)
      Unknown type conversion

### class UnavailableError(RiosError)
      A dependency is unavailable

### class VectorAttributeError(RiosError)
      Unable to find specified index in vector file

### class VectorGeometryTypeError(RiosError)
      Unexpected Geometry type

### class VectorLayerError(RiosError)
      Unable to find the specified layer

### class VectorProjectionError(RiosError)
      Vector projection does not match raster projection

### class VectorRasterizationError(RiosError)
      Rasterisation of Vector dataset failed

### class WorkerExceptionError(RiosError)
      A worker thread or process has raised an exception

### class WrongControlsObject(RiosError)
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
