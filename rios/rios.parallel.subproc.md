# rios.parallel.subproc
    Helper function for unpickling a user function and its
    parameters and running it. 
    
    Normally called from rios_subproc.py, but some parallel
    methods (MPI, multiprocessing etc) manage their own forking
    of the Python process so we must have this available as an 
    importable function.
    
    This file is entirely deprecated (as of version 2.0.0).

## Functions
### def runJob(inf, outf, inFileName=None)
        Run one job reading the pickle from inf
        and writing the output to outf.
        
        if inFileName is not None then then it
        will be deleted once it is read from.

## Values
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
