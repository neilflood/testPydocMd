# rios.parallel.jobmanager
    This file is entirely deprecated, as of version 2.0.0.
    
    Base class and sub-classes for managing parallel processing in RIOS. 
    
    It should be emphasised at the start that it is only worth using
    parallel processing in RIOS for tasks which are very computationally
    intensive, as there is significant overhead in setting up the sub-jobs. 
    Most image processing is I/O bound, and will not benefit from parallel 
    processing. 
    
    It should also be noted that the 'otherargs' parameter to :func:`rios.applier.apply`
    works for passing data into a user function, but updated data is not passed out at present.
    
    Also, certain functions on the 'info' parameter passed to the user function will not work.
    These are the functions that access the underlying GDAL dataset directly and include, 
    getGDALDatasetFor, getGDALBandFor, getNoDataValueFor and the global_* functions.
    
    The base class is :class:`rios.parallel.jobmanager.JobManager`. This is an abstract base class,
    and must be sub-classed before use. Any sub-class is intended to manage 
    processing of the user function in a set of sub-jobs, farming out 
    sub-jobs to process them all in parallel, and gathering up the 
    results, and re-combining into a single set of outputs. 
    
    Most of this work is handled in the base class, and should be generic
    for different methods of parallel processing. The reason for the
    sub-classes is to allow different approaches to be used, depending on 
    the system configuration. In particular, one can use a cluster batch 
    queue system such as PBS or SLURM to run sub-jobs as independent jobs, 
    allowing it to manage scheduling of jobs and resource management. 
    Alternatively, one can use MPI or Python's own multiprocessing module,
    if this is more appropriate for the system configuration available. 
    
    Sub-classes are provided for using PBS, SLURM, MPI, multiprocessing
    or Python's native subprocess module. Other sub-classes can be made
    as required, outside this module, and will be visible to the function::
    
        getJobManagerClassByName()
    
    which is the main function used for selecting which sub-class is required.
    
    The calling program controls the parallel processing through the
    ApplierControls() object. Normal usage would be as follows::
    
        from rios import applier
        controls = applier.ApplierControls()
        controls.setNumThreads(5)
    
    If a custom JobManager sub-class is used, its module should be imported 
    into the calling program (in order to create the sub-class), but its use 
    is selected using the same call to controls.setJobManagerType(), giving 
    the jobMgrType of the custom sub-class. 
    
    If $RIOS_DFLT_JOBMGRTYPE is set, this will be used as the default jobMgrType.
    This facilitates writing of application code which can run unmodified on 
    systems with different configurations. Alternatively, this can be set on
    the controls object, e.g.::
    
        controls.setJobManagerType('pbs')
        
    Environment Variables
    ---------------------
    
    +---------------------------------+-----------------------------------------------------------------+
    | Name                            | Description                                                     |
    +=================================+=================================================================+
    |RIOS_DFLT_JOBMGRTYPE             | Name string of default JobManager subclass                      |
    +---------------------------------+-----------------------------------------------------------------+
    |RIOS_PBSJOBMGR_QSUBOPTIONS       | String of commandline options to be used with PBS qsub.         |
    |                                 | Use this for things like walltime and queue name.               |
    +---------------------------------+-----------------------------------------------------------------+
    |RIOS_PBSJOBMGR_INITCMDS          | String of shell command(s) which will be executed               |
    |                                 | inside each PBS job, before executing the                       |
    |                                 | processing commands. Not generally required, but was            |
    |                                 | useful for initial testing.                                     |
    +---------------------------------+-----------------------------------------------------------------+
    |RIOS_SLURMJOBMGR_SBATCHOPTIONS   | String of commandline options to be used with SLURM             |
    |                                 | sbatch. Use this for things like walltime and queue name.       |
    |                                 | The output and error logs do not need to be specified - they    |
    |                                 | are set to temporary filenames by RIOS.                         |
    +---------------------------------+-----------------------------------------------------------------+
    |RIOS_SLURMJOBMGR_INITCMDS        | String of shell command(s) which will be executed               |
    |                                 | inside each SLURM job, before executing the                     |
    |                                 | processing commands. Not generally required, but was            |
    |                                 | useful for initial testing.                                     |
    +---------------------------------+-----------------------------------------------------------------+

## Classes
### class BlockAssociations
      Dummy class, to mimic applier.BlockAssociations, while avoiding circular imports. 

### class JobInfo
      Abstract base class for the information that needs to be passed to a job.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobInfo.getFunctionParams(self)
          Return the parameters to be passed to the actual function
          in the sub process. Should return a tuple.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobInfo.getFunctionResult(self, params)
          Return the parameter(s) that were modified
          by the function so they can be returned.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobInfo.prepareForPickling(self)
          Returns an instance of JobInfo to be pickled.
          Normally derived classes will just return 'self'
          but in some cases more complicated processing can
          happen

### class JobManager
      Manage breaking up of RIOS processing into sub-jobs, and farming them out. 
      
      Should be sub-classed to create new ways of farming out jobs. The sub-class 
      should at least over-ride the following abstract methods::
      
          startOneJob()
          waitOnJobs()
          gatherAllOutputs()
      
      More sophisticated sub-classes might also need to over-ride::
      
          startAllJobs()
      
      A sub-class must also include a class attribute called jobMgrType, which has 
      string value, which is the name used to select this sub-class. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.finalise(self)
          Do any tidy up at completion of image

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.gatherAllOutputs(self, jobIDlist)
          Gather up outputs from sub-jobs, and return a list of the
          outputs objects. 
          This is an abstract method, and must be over-ridden in a sub-class. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.runSubJobs(self, function, fnInputs)
          Take the given list of function arguments, run the given function 
          for each one, as a separate asynchronous job. 
          
          Returns a list of output BlockAssociations.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.setTempdir(self, tempdir)
          Directory to use for temporary files. This is generally set by apply(),
          using the one it has been given on the ApplierControls object. The
          default is '.'. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.startAllJobs(self, function, jobInputs)
          Start up all of the jobs processing blocks. Default implementation
          loops over the lists of jobs, starting each job separately. Keeps the
          first job aside, and runs it here before going off to wait for 
          the others. This means that the first element in the jobID list is not
          a jobID, but the results of the first sub-job. 
          
          jobInputs should be a list of JobInfo derived objects.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.startOneJob(self, userFunc, jobInfo)
          Start one job. Return a jobID object suitable for identifying the
          job, with all information required to wait for it, and 
          recover its output. This jobID is specific to the subclass. 
          
          This is an abstract method, and must be over-ridden in a sub-class.
          
          jobInfo should be a JobInfo derived object.        

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JobManager.waitOnJobs(self, jobIDlist)
          Wait until all the jobs in the given list have completed. This is
          an abstract method, and must be over-ridden in a sub-class. 

### class MpiJobManager(JobManager)
      Use MPI to run individual jobs. Requires mpi4py module. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MpiJobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MpiJobManager.gatherAllOutputs(self, jobIDlist)
          Gather up outputs from sub-jobs, and return a list of the
          outputs objects. Note that we assume that the first element of
          jobIDlist is actually an outputs object, from running the first sub-array
          in the current process. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MpiJobManager.startOneJob(self, userFunc, jobInfo)
          Start one job. Uses the MPI send call. 
          MPI does a certain level of pickling, but 
          we do our own here so that the function etc gets pickled.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MpiJobManager.waitOnJobs(self, jobIDlist)
          Can't actually wait on jobs with MPI, so we do nothing here.
          The waiting happens when we gatherAllOutputs() (and call MPI.recv)
          below.

### class MultiJobManager(JobManager)
      Use Python's standard multiprocessing module to run individual jobs.
      
      This JobManager sub-class should be used with caution, as it does not 
      involve any kind of load balancing, and all sub-processes simply run 
      concurrently. If you have enough spare cores and memory to do that, then
      no problem, but if not, you may clog the system. 
      
      This is much faster than the SubprocJobManager presumeably due to the
      custom data pickling that Python does.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MultiJobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MultiJobManager.gatherAllOutputs(self, jobIDlist)
          Gather up outputs from sub-jobs, and return a list of the
          outputs objects. Note that we assume that the first element of
          jobIDlist is actually an outputs object, from running the first sub-array
          in the current process. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MultiJobManager.startOneJob(self, userFunc, jobInfo)
          Start one job. Uses the multiprocessing.Pool.apply_async call. 
          This handles all the details of getting the data into the other
          process we we don't have to do any pickling here. 
          
          However we do call jobInfo.prepareForPickling() since
          multiprocessing has problems with the same things that the pickler
          does so we can assume the cleanup of the function here will suffice.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MultiJobManager.waitOnJobs(self, jobIDlist)
          Wait on all the jobs with AsyncResult.wait().
          The first element of jobIDlist is actually an outputs object
          so we ignore that.

### class PbsJobManager(JobManager)
      Use PBS to run individual jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PbsJobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PbsJobManager.gatherAllOutputs(self, jobIDlist)
          Gather up outputs from sub-jobs, and return a list of the
          outputs objects. Note that we assume that the first element of
          jobIDlist is actually an outputs object, from running the first sub-array
          in the current process. 
          
          The jobIDlist is a list of tuples whose second element is the name of 
          the output file containing the pickled outputs object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PbsJobManager.startOneJob(self, userFunc, jobInfo)
          Start one job. We create a shell script to submit to a PBS batch queue.
          When executed, the job will execute the rios_subproc.py command, giving
          it the names of two pickle files. The first is the pickle of all inputs
          (including the function), and the second is where it will write the 
          pickle of outputs. 
          
          Uses $RIOS_PBSJOBMGR_QSUBOPTIONS to pick up any desired options to the 
          qsub command. This should be used to control such things as requested 
          amount of memory or walltime for each job, which will otherwise be
          defaulted by PBS. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PbsJobManager.waitOnJobs(self, jobIDlist)
          Wait until all jobs in the given list have completed. The jobID values
          are tuples whose first element is a PBS job id string. We poll the PBS 
          queue until none of them are left in the queue, and then return. 
          
          Note that this also assumes the technique used by the default startAllJobs()
          method, of executing the first job in the current process, and so the first
          jobID is not a jobID but the results of that. Hence we do not try to wait on
          that job, but on all the rest. 
          
          Returns only when all the listed jobID strings are no longer found in the
          PBS queue. Currently has no time-out, although perhaps it should. 

### class SlurmJobManager(JobManager)
      Use SLURM to run individual jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SlurmJobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SlurmJobManager.gatherAllOutputs(self, jobIDlist)
          Gather up outputs from sub-jobs, and return a list of the
          outputs objects. Note that we assume that the first element of
          jobIDlist is actually an outputs object, from running the first sub-array
          in the current process. 
          
          The jobIDlist is a list of tuples whose second element is the name of 
          the output file containing the pickled outputs object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SlurmJobManager.startOneJob(self, userFunc, jobInfo)
          Start one job. We create a shell script to submit to a SLURM batch queue.
          When executed, the job will execute the rios_subproc.py command, giving
          it the names of two pickle files. The first is the pickle of all inputs
          (including the function), and the second is where it will write the 
          pickle of outputs. 
          
          Uses $RIOS_SLURMJOBMGR_SBATCHOPTIONS to pick up any desired options to the 
          sbatch command. This should be used to control such things as requested 
          amount of memory or walltime for each job, which will otherwise be
          defaulted by SLURM. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SlurmJobManager.waitOnJobs(self, jobIDlist)
          Wait until all jobs in the given list have completed. The jobID values
          are tuples whose first element is a SLURM job id string. We poll the SLURM 
          queue until none of them are left in the queue, and then return. 
          
          Note that this also assumes the technique used by the default startAllJobs()
          method, of executing the first job in the current process, and so the first
          jobID is not a jobID but the results of that. Hence we do not try to wait on
          that job, but on all the rest. 
          
          Returns only when all the listed jobID strings are no longer found in the
          SLURM queue. Currently has no time-out, although perhaps it should. 

### class SubprocJobManager(JobManager)
      Use Python's standard subprocess module to run individual jobs. 
      Passes input and output to/from the subprocesses using their 
      stdin/stdout. The command being executed is a simple main program
      which runs the user function on the given data, and passes back
      the resulting outputs object. 
      
      This JobManager sub-class should be used with caution, as it does not 
      involve any kind of load balancing, and all sub-processes simply run 
      concurrently. If you have enough spare cores and memory to do that, then
      no problem, but if not, you may clog the system. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocJobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocJobManager.gatherAllOutputs(self, jobIDlist)
          Gather up outputs from sub-jobs, and return a list of the
          outputs objects. Note that we assume that the first element of
          jobIDlist is actually an outputs object, from running the first sub-array
          in the current process. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocJobManager.startOneJob(self, userFunc, jobInfo)
          Start one job. We execute the rios_subproc.py command,
          communicating via its stdin/stdout. We give it the pickled
          function and all input objects, and we get back a pickled
          outputs object. 

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocJobManager.waitOnJobs(self, jobIDlist)
          Wait until all the jobs in the given list have completed. This
          implementation doesn't wait at all, because the subprocesses may
          block on writing their output to the stdout pipe. So, we do  
          nothing here, and actually wait on the read of the stdout from
          the subprocesses. 

## Functions
### def find_executable(executable)
        Our own version of distutils.spawn.find_executable that finds 
        the location of a script by trying all the paths in $PATH.
        Unlike distutils.spawn.find_executable, it does not add .exe
        to the script name under Windows.

### def getAvailableJobManagerTypes()
        Return a list of currently known job manager types

### def getJobManagerClassByType(jobMgrType)
        Return a sub-class of JobManager, selected by the type name
        given. 
        
        All sub-classes of JobManager will be searched for the 
        given jobMgrType string. 
            

### def getJobMgrObject(controls)
        Take an ApplierControls object and return a JobManager sub-class 
        object which meets the needs specified in the controls object. 
        If none is required, or none is available, then return None

### def multiUserFunc(userFunc, jobInfo)
        This function is run by the MultiJobManager to run
        one job. It runs the user function and then 
        returns the output block as multiprocessing.Pool 
        expects a function to behave.

## Values
    print_function = _Feature((2, 6, 0, 'alpha', 2), (3, 0, 0, 'alpha', 0), 1048576)
