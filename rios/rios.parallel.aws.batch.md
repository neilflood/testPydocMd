# rios.parallel.aws.batch
    This file is entirely deprecated, as of version 2.0.0.
    
    Module for implementing parallel with AWS Batch. 
    
    See the class AWSBatchJobManager for the implementation.
    
    Using AWS Services for parallel processing in RIOS
    ==================================================
    
    This directory holds implementations of per tile parallel processing
    using AWS services. Currently only AWS Batch is supported but it is
    intended that other services will be added in future.
    
    Refer to jobmanager.py for an overview of how RIOS handles parallel processing.
    
    AWS Batch
    =========
    
    Creating the infrastructure
    ---------------------------
    
    This implementation comes with a CloudFormation script (``templates/batch.yaml``)
    to create a separate VPC with all the infrastructure required. It is recommended
    to use the script `templates/createbatch.py` for the creation or modification (via the ``--modify``
    command line option) of this CloudFormation stack. There are also options for
    overriding some of the input parameters - see the output of `createbatch.py --help`
    for more information. NB: when running in a region that is NOT ap-southeast-2 you will
    need to update the availability zones (--az command line option).
    
    When you have completed processing you can run ``templates/deletebatch.py`` to delete
    all resources so you aren't paying for it. Note that you specify the region and stack
    name for this script via the RIOS_AWSBATCH_REGION and RIOS_AWSBATCH_STACK environment variables.
    
    Note that both ``createbatch.py`` and ``deletebatch.py`` have a ``--wait`` option that causes the
    script to keep running until creation/deletion is complete. 
    
    Creating the Docker image
    -------------------------
    
    AWS Batch requires you to provide a Docker image with the required software installed. 
    A `Dockerfile` is provided for this, but it it recommended that you use the `Makefile`
    to build the image as this handles the details of pulling the names out of the CloudFormation
    stack and creating a tar file of RIOS for copying into the Docker image. To build and push to 
    ECR simply run (being careful to set RIOS_AWSBATCH_REGION to the correct AWS region)::
    
        RIOS_AWSBATCH_REGION=ap-southeast-2 make
    
    By default this image includes GDAL, boto3 and RIOS. 
    
    Normally your script will need extra packages to run. You can specify the names of Ubuntu packages
    to also install with the environment variable `EXTRA_PACKAGES` like this::
    
        EXTRA_PACKAGES="python3-sklearn python3-skimage" RIOS_AWSBATCH_REGION=ap-southeast-2 make
    
    
    You can also use the `PIP_PACKAGES` environment variable to set the name of any pip packages like this::
    
        PIP_PACKAGES="pydantic python-dateutil" RIOS_AWSBATCH_REGION=ap-southeast-2 make
    
    You can also specify both if needed::
    
        EXTRA_PACKAGES="python3-sklearn python3-skimage" PIP_PACKAGES="pydantic python-dateutil"         RIOS_AWSBATCH_REGION=ap-southeast-2 make
    
    Setting up your main script
    ---------------------------
    
    To enable parallel processing using AWS Batch in your RIOS script you must import the batch module::
    
        from rios.parallel.aws import batch
    
    Secondly, you must set up an :class:`rios.applier.ApplierControls`
    object and pass it to :func:`rios.applier.apply`. On this
    object, you must make the following calls::
    
        controls.setNumThreads(4) # or whatever number you want
        controls.setJobManagerType('AWSBatch')
    
    Note that the number of AWS Batch jobs started will be (numThreads - 1) as one job is done by the main RIOS script.
    
    It is recommended that you run this main script within a container based on the one above. This reduces the likelihood
    of problems introduced by different versions of Python or other packages your script needs between the main RIOS
    script and the AWS Batch workers.
    
    To do this, create a `Dockerfile` like the one below (replacing `myscript.py` with the name of your script)::
    
        # Created by make command above
        FROM rios:latest
    
        COPY myscript.py /usr/local/bin
        RUN chmod +x /usr/local/bin/myscript.py
    
        ENTRYPOINT ["/usr/bin/python3", "/usr/local/bin/myscript.py"]
    
    Don't forget to pass in your ``AWS_ACCESS_KEY_ID`` and ``AWS_SECRET_ACCESS_KEY`` environment variables to this
    container when it runs (these variables are automatically set if running as a AWS Batch job but you'll
    need to set them otherwise).
    
    Also a good idea to pass in your RIOS_AWSBATCH_REGION and RIOS_AWSBATCH_STACK environment variables if the
    defaults have been overridden so that RIOS can find the CloudFormation stack.
    
    To also run you "main" Dockerfile as a batch job, push to the "RIOSecrMain" repository created by 
    ``templates/batch.yaml``. You can then submit jobs to the RIOSJobQueue using the RIOSJobDefinitionMain.
     

## Classes
### class AWSBatchException(Exception)
      Common base class for all non-exit exceptions.

### class AWSBatchJobManager(JobManager)
      Implementation of parallelisation via AWS Batch.
      This uses 2 SQS queues for communication between the
      'main' RIOS script and the subprocesses (which run on Batch)
      and an S3 bucket to hold the pickled data (which the SQS
      messages refer to).

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchJobManager.\_\_init\_\_(self, numSubJobs)
          numSubJobs is the number of sub-jobs

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchJobManager.finalise(self)
          Stop our AWS Batch jobs by sending a special message to the queue

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchJobManager.gatherAllOutputs(self, jobIDlist)
          Gather all the results. Checks the output SQS Queue

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchJobManager.startOneJob(self, userFunc, jobInfo)
          Start one sub job

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchJobManager.waitOnJobs(self, jobIDlist)
          Wait on all the jobs. Do nothing.

## Functions
### def getStackOutputs()
        Helper function to query the CloudFormation stack for outputs.
        
        Uses the RIOS_AWSBATCH_STACK and RIOS_AWSBATCH_REGION env vars to 
        determine which stack and region to query.

## Values
    REGION = 'ap-southeast-2'
    STACK_NAME = 'RIOS'
