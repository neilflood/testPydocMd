# rios.riostests.testbeforeclose
    Test the beforeCloseFunc callback.
    
    This is a function which is called just before the output file is closed,
    and is passed the still-open Dataset.

## Functions
### def beforeCloseNoArgs(ds)
        Called before closing, but takes no arguments other than ds

### def beforeCloseWithArgs(ds, name, val)
        Called before closing. Sets an arbitrary metadata item on the ds.

### def run()
        Run the test

### def userFunc(info, inputs, outputs)

## Values
    FIXED_NAME = 'fixedName'
    FIXED_VAL = 'fixedVal'
    TESTNAME = 'TESTBEFORECLOSE'
