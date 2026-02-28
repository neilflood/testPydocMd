# processinghistory.history
    Add processing history to a GDAL raster file
    
    This module attaches small text metadata to a GDAL raster file, using
    GDAL's arbitrary metadata API. The metadata is in the form of a dictionary
    of entries for things like the script which created it, a short description
    of what it is, and so on. In addition to that dictionary, there is also
    a copy of the history metadata for all the parent GDAL files that were
    inputs to creating the current file, so that the entire lineage is saved with
    the current file. This means the detail of its creation can be traced, even
    without access to the parent files.
    
    The metadata is stored as a JSON string in a single GDAL Metadata Item.
    
    Data Structures
    ---------------
    The whole processing history is stored as an instance of ProcessingHistory.
    This has two attributes, metadataByKey and parentsByKey, both of which are
    dictionary. Each of these is keyed by a tuple of the file name and the
    timestamp of that file. This means that references to a file in this context
    are referring to that file as created at that time, so that different versions
    of a file count as distinct entities. There are entries for all files in the
    lineage. The current file (i.e. the file containing this lineage) is keyed
    by a special key, so that the file's own name is not embedded inside itself.
    
    The metadataByKey dictionary has an entry for each file in the lineage, the
    value is that file's own metadata dictionary.
    
    The parentsByKey dictionary has an entry for each file in the lineage, the
    value being a list of keys of the parents of that file. This dictionary stores
    all the ancestry relationships for the whole lineage.
    
    History in VRT files
    --------------------
    A GDAL VRT file is handled as a somewhat special case. The component files
    of the VRT are treated as parents of the VRT (and there can be no other parents),
    and the history of those files is read directly from them, rather than being
    copied into the VRT. This is handled transparently, so that when history
    is read from the VRT, it appears to have all come from there. This allows the
    history of the components to be as dynamic as the data itself.

## Classes
### class ProcessingHistory
      Hold whole all ancestry and metadata for a single file

#### &nbsp;&nbsp;&nbsp;&nbsp; ProcessingHistory.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; ProcessingHistory.addParentHistory(self, parentfile)
        Add history from parent file to self

#### &nbsp;&nbsp;&nbsp;&nbsp; ProcessingHistory.findKeyByFile(self, filename)
        Return a list of all full keys from self.metadataByKey which match the
        given filename. Normally this is just a single key, so the list has
        only one element, but this should be checked.

#### &nbsp;&nbsp;&nbsp;&nbsp; ProcessingHistory.fromJSON(jsonStr)
        Return a ProcessingHistory object from the given JSON string

#### &nbsp;&nbsp;&nbsp;&nbsp; ProcessingHistory.toJSON(self)
        Return a JSON representation of the current ProcessingHistory

### class ProcessingHistoryError
      Generic exception for ProcessingHistory

## Functions
### def makeAutomaticFields()
        Generate a dictionary populated with all the fields which are automatically
        set.

### def makeProcessingHistory(userDict, parents)
        Make the full processing history. Returns a dictionary with all metadata
        and parentage relationships.

### def readHistoryFromFile(filename=None, gdalDS=None)
        Read processing history from file.
        
        File to read can be specified as either a filename, or an open GDAL
        Dataset object.

### def versionFromDistribution(modname)
        If possible, deduce a package version number for the given
        module/package name, using distribution metadata.
        
        If available, return a tuple of (distributionName, versionStr)
        Note that the distribution name may not be the same as the
        module or package name.
        
        If unavailable for any reason, return (None, None).

### def writeHistoryToFile(userDict={}, parents=[], *, filename=None, gdalDS=None)
        Make the full processing history and save to the given file.
        
        File can be specified as either a filename string or an open GDAL Dataset

## Values
    AUTOENVVARSLIST_NAME = 'HISTORY_ENVVARS_TO_AUTOINCLUDE'
    CURRENTFILE_KEY = 'CURRENTFILE'
    HAVE_IMPLIB_METADATA = True
    METADATA_BY_KEY = 'metadataByKey'
    METADATA_GDALITEMNAME = 'ProcessingHistory'
    NO_TIMESTAMP = 'UnknownTimestamp'
    PARENTS_BY_KEY = 'parentsByKey'
    TIMESTAMP = 'timestamp'
