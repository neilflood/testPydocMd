# rios.computemanager
    The ComputeWorkerManager abstract base class is for creating and managing
    a set of compute workers. Each different computeWorkerKind is implemented
    as a concrete subclass of this. All these classes are found in this module.

## Classes
### class AWSBatchComputeWorkerMgr(ComputeWorkerManager)
      Manage compute workers using AWS Batch.
      
      Obsolete, and likely to be deprecated. See ECSComputeWorkerMgr instead.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchComputeWorkerMgr.getStackOutputs(self)
          Helper function to query the CloudFormation stack for outputs.
          
          Uses the RIOS_AWSBATCH_STACK and RIOS_AWSBATCH_REGION env vars to
          determine which stack and region to query.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchComputeWorkerMgr.shutdown(self)
          Shut down the job pool

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AWSBatchComputeWorkerMgr.startWorkers(self, numWorkers=None, userFunction=None, infiles=None, outfiles=None, otherArgs=None, controls=None, blockList=None, inBlockBuffer=None, outBlockBuffer=None, workinggrid=None, allInfo=None, singleBlockComputeWorkers=False, tmpfileMgr=None, haveSharedTemp=True, exceptionQue=None)
          Start <numWorkers> AWS Batch jobs to process blocks of data

### class ClassicBatchComputeWorkerMgr(ComputeWorkerManager)
      Manage compute workers using a classic batch queue, notably
      PBS or SLURM. Initially constructed with computeWorkerKind = None,
      one must then assign computeWorkerKind as either CW_PBS or CW_SLURM
      before use.
      
      Will make use of the computeWorkerExtraParams argument to ConcurrencyStyle,
      if given, but this is optional. If given, it should be a dictionary, see
      :doc:`concurrency` for details.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.beginScript(self, logfile, workerID)
          Return list of initial script commands, depending on
          whether we are PBS or SLURM

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.checkBatchSystemAvailable(self)
          Check whether the selected batch queue system is available.
          If not, raise UnavailableError

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.findExtraErrors(self)
          Look for errors in the log files. These would be errors which were
          not reported via the data channel

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.findLine(linelist, s)
          Find the first line which begins with the given string.
          Return the index of that line, or None if not found.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.getJobId(self, stdout)
          Extract the jobId from the string returned when the job is
          submitted, depending on whether we are PBS or SLURM

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.getQlistHeaderCount(self)
          Number of lines to skip at the head of the qlist output

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.getQueueCmd(self)
          Return the command name for listing the current jobs in the
          batch queue, depending on whether we are PBS or SLURM. Return
          as a list of words, ready to give to Popen.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.getSubmitCmd(self)
          Return the command name for submitting a job, depending on
          whether we are PBS or SLURM. Return as a list of words,
          ready to give to Popen.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.shutdown(self)
          Shutdown the compute manager. Wait on batch jobs, then
          shut down the data channel

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.startWorkers(self, numWorkers=None, userFunction=None, infiles=None, outfiles=None, otherArgs=None, controls=None, blockList=None, inBlockBuffer=None, outBlockBuffer=None, workinggrid=None, allInfo=None, singleBlockComputeWorkers=False, tmpfileMgr=None, haveSharedTemp=True, exceptionQue=None)
          Start <numWorkers> PBS or SLURM jobs to process blocks of data

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.waitOnJobs(self)
          Wait for all batch jobs to complete

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassicBatchComputeWorkerMgr.worker(self, workerID, tmpfileMgr)
          Assemble a worker job and submit it to the batch queue

### class ComputeWorkerManager(ABC)
      Abstract base class for all compute-worker manager subclasses
      
      A subclass implements a particular way of managing RIOS
      compute-workers. It should over-ride all abstract methods given here.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ComputeWorkerManager.getWorkerName(self, workerID)
          Return a string which uniquely identifies each work, including
          the jobName, if given.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ComputeWorkerManager.makeOutObjList(self)
          Make a list of all the objects the workers put into outqueue
          on completion

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ComputeWorkerManager.setJobName(self, jobName)
          Sets the job name string, which is made available to worker
          processes. Defaults to None, and has only cosmetic effects.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ComputeWorkerManager.setupNetworkCommunication(self, userFunction, infiles, outfiles, otherArgs, controls, workinggrid, allInfo, blockList, numWorkers, inBlockBuffer, outBlockBuffer, forceExit, exceptionQue, workerBarrier)
          Set up the standard methods of network communication between
          the workers and the main thread. This is expected to be the
          same for all workers running on separate machines from the
          main thread.
          
          Creates the dataChan and outqueue attributes.
          
          This routine is not needed for the Threads subclass, because it
          does not use the network versions of these communications.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ComputeWorkerManager.shutdown(self)
          Shutdown the computeWorkerManager

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ComputeWorkerManager.startWorkers(self, numWorkers=None, userFunction=None, infiles=None, outfiles=None, otherArgs=None, controls=None, blockList=None, inBlockBuffer=None, outBlockBuffer=None, workinggrid=None, allInfo=None, singleBlockComputeWorkers=False, tmpfileMgr=None, haveSharedTemp=True, exceptionQue=None)
          Start the specified compute workers

### class ECSComputeWorkerMgr(ComputeWorkerManager)
      Manage compute workers using Amazon AWS ECS
      
      New in version 2.0.7
      
      Requires some extra parameters in the ConcurrencyStyle constructor
      (computeWorkerExtraParams), in order to configure the AWS infrastructure.
      This class provides some helper functions for creating these for
      various use cases.
      
      When creating a private cluster of EC2 instances, these are automatically
      tagged with some AWS tags. See `Concurrency <concurrency.html>`_ doc page
      for details.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.checkTaskErrors(self)
          Check for errors in any of the worker tasks, and report to stderr.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.createCluster(self)
          If requested to do so, create an ECS cluster to run on.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.createTaskDef(self)
          If requested to do so, create a task definition for the worker tasks

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.getClusterInstanceCount(self, clusterName)
          Query the given cluster, and return the number of instances it has. If the
          cluster does not exist, return None.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.getClusterTaskCount(self)
          Query the cluster, and return the number of tasks it has.
          This is the total of running and pending tasks.
          If the cluster does not exist, return None.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.makeExtraParams_Fargate(jobName=None, containerImage=None, taskRoleArn=None, executionRoleArn=None, subnets=None, subnet=None, securityGroups=None, cpu='0.5 vCPU', memory='1GB', cpuArchitecture=None, cloudwatchLogGroup=None, tags=None)
          Helper function to construct a minimal computeWorkerExtraParams
          dictionary suitable for using ECS with Fargate launchType, given
          just the bare essential information.
          
          Returns a Python dictionary.
          
          jobName : str
              Arbitrary string, optional. If given, this name will be incorporated
              into some AWS/ECS names for the compute workers, including the
              container name and the task family name.
          containerImage : str
              Required. URI of the container image to use for compute workers. This
              container must have RIOS installed. It can be the same container as
              used for the main script, as the entry point is over-written.
          executionRoleArn : str
              Required. ARN for an AWS role. This allows ECS to use AWS services on
              your behalf. A good start is a role including
              AmazonECSTaskExecutionRolePolicy, which allows access to ECR
              container registries and CloudWatch logs.
          taskRoleArn : str
              Required. ARN for an AWS role. This allows your code to use AWS
              services. This role should include policies such as AmazonS3FullAccess,
              covering any AWS services your compute workers will need.
          subnet : str
              Required. Subnet ID string associated with the VPC in which
              workers will run.
          subnets : list of str
              Deprecated. List of subnet ID strings associated with the VPC in which
              workers will run. This is an alternative to specifying a single
              subnet, but is deprecated, and should not be used. As far as we know,
              there is no good reason to spread workers across multiple subnets.
          securityGroups : list of str
              Required. List of security group IDs associated with the VPC.
          cpu : str
              Number of CPU units requested for each compute worker, expressed in
              AWS's own units. For example, '0.5 vCPU', or '1024' (which
              corresponds to the same thing). Both must be strings. This helps
              Fargate to select a suitable VM instance type (see below).
          memory : str
              Amount of memory requested for each compute worker, expressed in MiB,
              or with a units suffix. For example, '1024' or its equivalent '1GB'.
              This helps Fargate to select a suitable VM instance type (see below).
          cpuArchitecture : str
              If given, selects the CPU architecture of the hosts to run worker on.
              Can be 'ARM64', defaults to 'X86_64'.
          cloudwatchLogGroup : str or None
              Optional. Name of CloudWatch log group. If not None, each worker
              sends a log stream of its stdout & stderr to this log group. The
              group should already exist. If None, no CloudWatch logging is done.
              Intended for tracking obscure problems, rather than to use permanently.
          tags: dict or None
              Optional. If specified this needs to be a dictionary of key/value
              pairs which will be turned into AWS tags. These will be added to
              the ECS cluster, task definition and tasks. The keys and values
              must all be strings. Requires ``ecs:TagResource`` permission.
          
          Only certain combinations of cpu and memory are allowed, as these are used
          by Fargate to select a suitable VM instance type. See ESC.Client.run_task()
          documentation for further details.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.makeExtraParams_PrivateCluster(jobName=None, numInstances=None, ami=None, instanceType=None, containerImage=None, taskRoleArn=None, executionRoleArn=None, subnet=None, securityGroups=None, instanceProfileArn=None, memoryReservation=1024, cloudwatchLogGroup=None, tags=None)
          Helper function to construct a basic computeWorkerExtraParams
          dictionary suitable for using ECS with a private per-job cluster,
          given just the bare essential information.
          
          Returns a Python dictionary.
          
          jobName : str
              Arbitrary string, optional. If given, this name will be incorporated
              into some AWS/ECS names for the compute workers, including the
              container name and the task family name.
          numInstances : int
              Number of VM instances which will comprise the private ECS cluster.
              The RIOS compute workers will be distributed across these, so it
              makes sense to have the same number of instances, i.e. one worker
              on each instance.
          ami : str
              Amazon Machine Image ID string. This should be for an ECS-Optimized
              machine image, either as supplied by AWS, or custom-built, but it must
              have the ECS Agent installed. An example would
              be "ami-00065bb22bcbffde0", which is an AWS-supplied ECS-Optimized
              image.
          instanceType : str
              The string identifying the instance type for the VM instances which
              will make up the ECS cluster. An example would be "a1.medium".
          containerImage : str
              Required. URI of the container image to use for compute workers. This
              container must have RIOS installed. It can be the same container as
              used for the main script, as the entry point is over-written.
          executionRoleArn : str
              Required. ARN for an AWS role. This allows ECS to use AWS services on
              your behalf. A good start is a role including
              AmazonECSTaskExecutionRolePolicy, which allows access to ECR
              container registries and CloudWatch logs.
          taskRoleArn : str
              Required. ARN for an AWS role. This allows your code to use AWS
              services. This role should include policies such as AmazonS3FullAccess,
              covering any AWS services your compute workers will need.
          subnet : str
              Required. A subnet ID string associated with the VPC in which
              workers will run.
          securityGroups : list of str
              Required. List of security group IDs associated with the VPC.
          instanceProfileArn : str
              The IamInstanceProfile ARN to use for the VM instances. This should
              include AmazonEC2ContainerServiceforEC2Role policy, which allows the
              instances to be part of an ECS cluster.
          memoryReservation : int
              Optional. Memory (in MiB) reserved for the containers in each
              compute worker. This should be small enough to fit well inside the
              memory of the VM on which it is running. Often best to leave this
              as default until out-of-memory errors occur, then increase.
          cloudwatchLogGroup : str or None
              Optional. Name of CloudWatch log group. If not None, each worker
              sends a log stream of its stdout & stderr to this log group. The
              group should already exist. If None, no CloudWatch logging is done.
              Intended for tracking obscure problems, rather than to use permanently.
          tags: dict or None
              Optional. If specified this needs to be a dictionary of key/value
              pairs which will be turned into AWS tags. These will be added to
              the ECS cluster, task definition and tasks, and the EC2 instances.
              The keys and values must all be strings. Requires ``ecs:TagResource``
              permission.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.makeJobIDstr(jobName)
          Make a job ID string to use in various generate names. It is unique to
          this run, and also includes any human-readable information available

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.runInstances(self, numWorkers)
          If requested to do so, run the instances required to populate
          the cluster

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.shutdown(self)
          Shut down the workers

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.shutdownCluster(self)
          Shut down the ECS cluster, if one has been created

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.startWorkers(self, numWorkers=None, userFunction=None, infiles=None, outfiles=None, otherArgs=None, controls=None, blockList=None, inBlockBuffer=None, outBlockBuffer=None, workinggrid=None, allInfo=None, singleBlockComputeWorkers=False, tmpfileMgr=None, haveSharedTemp=True, exceptionQue=None)
          Start <numWorkers> ECS tasks to process blocks of data

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.waitClusterInstanceCount(self, clusterName, endInstanceCount)
          Poll the given cluster until the instanceCount is equal to the
          given endInstanceCount

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ECSComputeWorkerMgr.waitClusterTasksFinished(self)
          Poll the given cluster until the number of tasks reaches zero

### class SubprocComputeWorkerManager(ComputeWorkerManager)
      Purely for testing, not for normal use.
      
      This class manages compute workers run through subprocess.Popen.
      This is not normally any improvement over using CW_THREADS, and
      should be avoided. I am using this purely as a test framework
      to emulate the batch queue types of compute worker, which are
      similarly disconnected from the main process, so I can work out the
      right mechanisms to use for exception handling and such like,
      and making sure the rios_computeworker command line works.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocComputeWorkerManager.findExtraErrors(self)
          Check for errors in any worker stderr. These would be errors not
          reported via the data channel

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocComputeWorkerManager.shutdown(self)
          Shutdown the compute manager. Wait on batch jobs, then
          shut down the data channel

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocComputeWorkerManager.startWorkers(self, numWorkers=None, userFunction=None, infiles=None, outfiles=None, otherArgs=None, controls=None, blockList=None, inBlockBuffer=None, outBlockBuffer=None, workinggrid=None, allInfo=None, singleBlockComputeWorkers=False, tmpfileMgr=None, haveSharedTemp=True, exceptionQue=None)
          Start the specified compute workers

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocComputeWorkerManager.waitOnJobs(self)
          Wait for all worker subprocesses to complete

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubprocComputeWorkerManager.worker(self, workerID)
          Start one worker

### class ThreadsComputeWorkerMgr(ComputeWorkerManager)
      Manage compute workers using threads within the current process.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ThreadsComputeWorkerMgr.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ThreadsComputeWorkerMgr.shutdown(self)
          Shut down the thread pool

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ThreadsComputeWorkerMgr.startWorkers(self, numWorkers=None, userFunction=None, infiles=None, outfiles=None, otherArgs=None, controls=None, blockList=None, inBlockBuffer=None, outBlockBuffer=None, workinggrid=None, allInfo=None, singleBlockComputeWorkers=False, tmpfileMgr=None, haveSharedTemp=True, exceptionQue=None)
          Start <numWorkers> threads to process blocks of data

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ThreadsComputeWorkerMgr.worker(self, userFunction, infiles, outfiles, otherArgs, controls, allInfo, workinggrid, blockList, inBlockBuffer, outBlockBuffer, outqueue, workerID, exceptionQue)
          This function is a worker for a single thread, with no reading
          or writing going on. All I/O is via the inBlockBuffer and
          outBlockBuffer objects.

## Functions
### def getComputeWorkerManager(cwKind)
        Returns a compute-worker manager object of the requested kind.

## Values
    CW_AWSBATCH = 'CW_AWSBATCH'
    CW_ECS = 'CW_ECS'
    CW_NONE = 'CW_NONE'
    CW_PBS = 'CW_PBS'
    CW_SLURM = 'CW_SLURM'
    CW_SUBPROC = 'CW_SUBPROC'
    CW_THREADS = 'CW_THREADS'
