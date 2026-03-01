# processinghistory.cmdline.historyview
    Command line tool to view processing history in simple text form.

## Classes
### class HistoryviewTextError(Exception)
      Common base class for all non-exit exceptions.

## Functions
### def displayDict(d, cmdargs)
        Display the given dictionary in simple text table form

### def displayParents(childKey, procHist)
        Display the given list of parents in simple text form

### def displayWholeLineage(procHist)
        Display parent relationships for whole lineage

### def findAncestorKey(procHist, ancestor)
        Find the key for the given ancestor. Ancestor is a string, and can
        be either a filename or a string representation of a key tuple.
        
        If ancestor is already a key tuple string, this is eval-ed and returned
        as a tuple.
        
        If no key matches the filename, an exception is raised. If multiple
        keys match, they are printed to stdout and None is returned. Otherwise,
        return a single key tuple.

### def getCmdargs()
        Get command line arguments

### def mainCmd()
        Main

