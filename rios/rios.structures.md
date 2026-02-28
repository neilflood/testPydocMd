# rios.structures
    Many of the major data structures to support the new applier parallel
    architecture.

## Classes
### class ApplierBlockDefn
      Defines a single block of the working grid. Is hashable and ordered.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierBlockDefn.\_\_init\_\_(self, top, left, nrows, ncols)
        Initialize self.  See help(type(self)) for accurate signature.

### class ApplierReturn
      Hold all objects returned by the applier.apply() function. Some fields
      are useful operationally, while others are purely for information about
      the inner workings of RIOS.
      
      Fields
          timings: an instance of :class:`rios.structures.Timers`
      
          otherArgsList: list of :class:`rios.structures.OtherInputs`
              By default, there is only one element, and it is the same as
              the one passed in to apply(). However, these objects are not
              thread-safe, so when using multiple compute workers, each worker
              has its own copy of otherArgs, which it can modify independently.
              These copies are then collected up again after all workers have
              finished, and the list of these is made available on this return
              object. The user is then free to merge these in whatever way is
              suitable.
      
          workinggrid: an instance of :class:`rios.pixelgrid.PixelGridDefn`
              The final working grid
      
          singlepassMgr: an instance of :class:`rios.calcstats.SinglePassManager`
              The object which manages the workings of single-pass output
              operations for pyramids (overviews), basic statistics, and
              histograms.

#### &nbsp;&nbsp;&nbsp;&nbsp; ApplierReturn.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

### class BlockAssociations
      Container class to hold raster arrays for a single block.
      
      If the constructor is given a FilenameAssociations object,
      it populates the BlockAssociations object with None to match
      the same structure of names and sequences. Otherwise the object
      is empty.
      
      This object can be indexed, using a tuple of
      (symbolicName, sequenceNumber) as a key. This can be used for
      both getting and setting an array of data on the object. If the
      symbolicName corresponds to a list, then the sequenceNumber should
      be an integer index value, but if the symbolicName corresponds to
      a single filename, then the sequenceNumber should be None.
      
      In order to set a value via indexing, the object must have been
      created from a corresponding FilenameAssociations object, which
      determines the structure of the object (i.e. the valid names and
      index values).

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockAssociations.\_\_init\_\_(self, fnameAssoc=None)
        Initialize self.  See help(type(self)) for accurate signature.

### class BlockBuffer
      Buffer of blocks of data which have been read in. Blocks
      may exist but be incomplete, as individual inputs are
      added to them. This structure is shared by all read workers
      within a given process, so includes locking mechanisms to
      make it thread-safe.

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.\_\_init\_\_(self, filenameAssoc, numWorkers, insertTimeout, popTimeout, bufferTypeName)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.addBlockData(self, blockDefn, name, seqNum, arr)
        Use when building up blocks one array at a time

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.insertCompleteBlock(self, blockDefn, blockData)
        Use when inserting a complete BlockAssociations object at once

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.popCompleteBlock(self, blockDefn)
        Returns the BlockAssociations object for the given blockDefn,
        and removes it from the buffer

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.popNextBlock(self)
        Pop the next completed block from the buffer, without regard to
        which block it is. Return a tuple of objects
        
            (ApplierBlockDefn, BlockAssociations)

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.timeoutName(self, timeoutType)
        Deduce the name of the relevant timeout, using the
        bufferTypeName given to the constructor, and the type of timeout

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBuffer.waitCompletion(self, blockDefn, timeout=None)
        Wait until the given block is complete

### class BlockBufferValue
      Used to hold a BlockAssociations object, along with relevant information
      about its completeness, and locking to ensure thread-safety. An instance
      of this is used for each BlockAssociations object stored in a BlockBuffer.

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBufferValue.\_\_init\_\_(self, filenameAssoc=None, blockData=None)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBufferValue.addData(self, name, seqNum, arr)

#### &nbsp;&nbsp;&nbsp;&nbsp; BlockBufferValue.complete(self)

### class ConcurrencyStyle
      Class to hold all parameters associated with the different styles
      of concurrency.
      
      By default, concurrency is switched off. Concurrency can be
      switched on in two areas - reading, and computation.
      
      The writing of output blocks is always done one block at a time,
      because GDAL does not support write concurrency, but it can
      overlap with the reading and/or computation of later blocks.
      
      Concurrency in computation is supported at the block level. Each
      block of data is computed by a single call to the user function,
      and these calls can be distributed to a number of compute workers,
      with a choice of different distributed paradigms.
      
      Concurrency in computation is likely to be of benefit only when
      the user function is carrying out a computationally intensive
      task. If the computation is very simple and fast, then I/O is likely
      to be the main bottleneck, and compute concurrency will provide
      little or no speedup.
      
      Concurrency in reading is supported in two ways. Firstly,
      a pool of threads can read single blocks of data from individual
      files, which are placed in a buffer, ready for use in computation. This
      allows for reading multiple input files at the same time, and for
      reading ahead of the computation. Secondly, each compute worker can
      do its own reading (assuming that the input files are accessible
      to the compute workers), and the data is consumed directly by each
      worker. If desired, these two read paradigms can be used together,
      allowing each compute worker to run its own pool of read threads.
      
      Concurrency in reading is mainly beneficial when the input data is
      on a device which scales well with high load, such as a cluster
      disk array. There is even more benefit when the device also has high
      latency, such as an AWS S3 bucket. If input data is on a single local
      disk, then adding a single read worker will allow reading to overlap
      with computation, but more read workers are unlikely to improve on the
      caching and buffering already provided by a sensible operating system.
      
      Note that not all possible combinations of parameters are supported,
      and some combinations make no sense at all.
      
      Read Concurrency
          numReadWorkers: int
              The number of read workers. A value of 0 means that reading
              happens sequentially within the main processing loop. A value
              greater than zero will start this many independent threads
              within the main RIOS process to read blocks of data, placing
              them in a buffer, ready for the computation to use them. This
              can be used independently of whether any compute workers are
              used.
              Also see computeWorkersRead, below, for the interaction with
              compute workers.
      
      Compute Concurrency
          computeWorkerKind: One of {CW_NONE, CW_THREADS, CW_ECS, CW_PBS, CW_SLURM,
              CW_AWSBATCH, CW_SUBPROC}.
      
              Selects the paradigm used to distribute compute workers.
              The CW_THREADS option means a pool of compute threads
              running within the same process as the rest of RIOS. This is
              almost certainly the best option to start exploring compute
              concurrency in RIOS.
      
              The CW_ECS option is the most sophisticated. The compute workers
              run as separate containerized tasks on an AWS ECS cluster, with
              options to control how those ECS tasks are launched.
      
              The CW_PBS, CW_SLURM and CW_AWSBATCH options all refer to different
              batch queue systems, so that compute workers can run as jobs
              on the batch queue. In those cases, not only do the workers
              run as separate processes, they may also be running on separate
              machines. All these options are currently somewhat experimental,
              and should be treated with caution.
      
              A value of CW_NONE means that computation happens sequentially
              within the main processing loop.
      
              See the `Concurrency <concurrency.html>`_ doc page
              for a deeper discussion on suitable use of the different
              kinds of compute worker.
          computeWorkerExtraParams: dict
              Any extra parameters required by a particular computeWorkerKind.
              This is a Python dictionary, and its contents are interpreted by
              the requested computeWorkerKind. See the documentation
              for the kind in question. Default is None.
              
          numComputeWorkers: int
              The number of distinct compute workers. If zero, then
              computation happens within the main processing loop.
          computeWorkersRead: bool
              The default is None, which means that a sensible True/False
              choice will be made by the selected computeWorkerKind. In most
              cases, this should be left to default.
      
              If True, then each compute worker does its own reading,
              possibly with its own pool of read worker threads
              (<numReadWorkers> threads for each compute worker). This
              is likely to be a good option with any computeWorkerKind
              which involves compute workers running on separate machines.
      
              If False, then all reading is done by the main RIOS process
              (possibly using one or more read workers) and data is sent
              directly to each compute worker. False (or None) is required
              for CW_THREADS compute workers, but may also be useful in cases
              when compute workers are on separate machines on an internal
              network, but input files are not accessible to those machines,
              and must be read by the main RIOS process.
          singleBlockComputeWorkers: bool
              This applies only to the batch queue paradigms. In some
              batch configurations, it is advantageous to run many small,
              quick jobs. If singleBlockComputeWorkers is True, then
              numComputeWorkers is ignored, and a batch job is generated
              for every block to be processed. It is then up to the batch
              queue system's own load balancing to decide how many jobs are
              running concurrently. This is likely to be of most benefit
              for large shared PBS and SLURM batch queues with plenty of
              available nodes. However, it should be used with caution, with
              regard to the timeouts which could occur (see below).
          haveSharedTemp: bool
              If True, then the compute workers are all able to see a shared
              temporary directory. This is ignored for some computeWorkerKinds,
              but for the PBS and SLURM kinds, the temp dir is used to share a
              small text file (only readable by the user) giving the network
              address for all other communication. If False, then the address
              information is passed on the command line of the batch jobs,
              which is publicly visible and so less secure.
      
      Buffering Timeouts (seconds)
          The block buffers have several timeout periods defined, with default
          values. These can be over-ridden here. Mostly these timeouts should
          not be reached, but it is vital to have them. In the event of errors
          in one or more workers, whatever is waiting for that worker to respond
          would otherwise wait forever, with no explanation. The times given 
          are all in terms of the wait for a single block of data to become 
          available. For very slow input devices or very long computations, 
          they may need to be increased, but generally, if a timeout occurs, 
          one should first rule out any errors in the relevant workers before 
          increasing the timeout period.
      
          These timeout values can each be set to None, in which case the
          corresponding wait will never timeout.
      
          readBufferInsertTimeout: int
              Time to wait to insert a new (empty) block into the read buffer
          readBufferPopTimeout: int
              Time to wait to pop a complete block out of the read buffer,
              for a compute worker to use
          computeBufferInsertTimeout: int
              Time to wait to insert a completed block into the compute buffer,
              ready for the writing thread
          computeBufferPopTimeout: int
              Time to wait to pop a block out of the compute buffer, to
              write it to the outfiles
      
      computeBarrierTimeout: int
          This applies only to the batch-oriented compute worker types, and
          only when singleBlockComputeWorkers is False. For any other styles
          it is ignored. Processing is blocked until all batch compute workers
          have had a chance to start, after which everything proceeds. The
          wait at this barrier will timeout after this many seconds.

#### &nbsp;&nbsp;&nbsp;&nbsp; ConcurrencyStyle.\_\_init\_\_(self, numReadWorkers=0, numComputeWorkers=0, computeWorkerKind='CW_NONE', computeWorkerExtraParams=None, computeWorkersRead=None, singleBlockComputeWorkers=False, haveSharedTemp=True, readBufferInsertTimeout=10, readBufferPopTimeout=10, computeBufferInsertTimeout=10, computeBufferPopTimeout=20, computeBarrierTimeout=600)
        Initialize self.  See help(type(self)) for accurate signature.

### class FilenameAssocIterator
      Separate class for the iterator of a FilenameAssociations object.
      
      Using a separate class allows us to maintain the iteration state here,
      without putting these extra variables onto the original FilenameAssociations
      object. Not sure how important this is, but it is an approach originally
      suggested in the old Python docs. In later Python versions, this approach is
      no longer explicitly suggested in the docs, but seems to have some merit,
      so we still do it this way.
      
      When one iterates a FilenameAssociations object, the loop is actually
      iterating on a new instance of this class, and the original object is thus
      left untouched.

#### &nbsp;&nbsp;&nbsp;&nbsp; FilenameAssocIterator.\_\_init\_\_(self, infiles)
        Initialize self.  See help(type(self)) for accurate signature.

### class FilenameAssociations
      Class for associating external image filenames with internal
      names, which are then the same names used inside a function given
      to the :func:`rios.applier.apply` function.
      
      Each attribute created on this object should be a filename, or a
      list of filenames. The corresponding attribute names will appear
      on the 'inputs' or 'outputs' objects inside the applied function.
      Each such attribute will be an image data block or a list of image
      data blocks, accordingly.
      
      This object can be used as an iterator. Each iteration will return
      a tuple of (symbolicName, sequenceNumber, filename). The symbolicName
      is the name for each attribute on the object. If this corresponds
      to a single filename, then the sequenceNumber is None. If it is a list,
      then the iterator will return each of the files in the list as a new
      iteration, with the sequenceNumber being the index in the list. In this
      way, a single loop is able to iterate through all of the files defined
      on the object, with full information about where they are found.
      
      The object can also be indexed, using a tuple of
      (symbolicName, sequenceNumber) as an index. The value at that index
      is the corresponding filename. If seqNum is None, or the index is just
      the symbolicName string instead of a tuple, then the index operation
      returns the full entry for that symbolicName, which may be either a
      single filename or a list of filenames.
      
      Indexing is read-only, and cannot be used to set filenames.

### class NetworkDataChannel
      A network-visible channel to serve out all the required information to
      a group of RIOS compute workers.
      
      The channel has several major attributes.
      
          workerInitData
              a dictionary of objects which are used to initialize each
              worker. This read-only, and cannot be modified by the workers.
          inBlockBuffer
              None, if compute workers are doing their own reading,
              otherwise it is a BlockBuffer supplying input data to the
              compute workers.
          outBlockBuffer
              a BlockBuffer where completed 'outputs' objects are
              placed, ready for writing.
          outqueue
              a Queue. It is used for any non-pixel output data
              coming from each compute worker, such as modified otherArgs
              objects. Anything in this queue will be collected up by the
              main thread after all compute workers have completed.
          forceExit
              An Event object. If set, this signals that workers should exit
              as soon as possible
          exceptionQue
              A Queue. Any exceptions raised in the worker are caught and put
              into this queue, to be dealt with in the main thread.
          workerBarrier
              A Barrier object. For the relevant compute worker kinds, all
              workers will wait at this barrier, as will the main thread,
              so that no processing starts until all compute workers are
              ready to work.
      
      If the constructor is given these major objects as arguments, then this
      is the server of these objects, and they are served to the network on
      a selected port number. The address of this server is available on the
      instance as hostname, portnum and authkey attributes. The server will
      create its own thread in which to run.
      
      A client instance can be created by giving the constructor the hostname,
      port number and authkey (obtained from the server object). This will then
      connect to the server, and make available the data attributes as given.
      
      The server must be shut down correctly, and so the shutdown() method
      should always be called explicitly.

#### &nbsp;&nbsp;&nbsp;&nbsp; NetworkDataChannel.\_\_init\_\_(self, workerInitData=None, inBlockBuffer=None, outBlockBuffer=None, forceExit=None, exceptionQue=None, workerBarrier=None, hostname=None, portnum=None, authkey=None)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; NetworkDataChannel.addressStr(self)
        Return a single string encoding the network address of this channel

#### &nbsp;&nbsp;&nbsp;&nbsp; NetworkDataChannel.shutdown(self)
        Shut down the NetworkDataChannel in the right order. This should always
        be called explicitly by the creator, when it is no longer
        needed. If left to the garbage collector and/or the interpreter
        exit code, things are shut down in the wrong order, and the
        interpreter hangs on exit.
        
        I have tried __del__, also weakref.finalize and atexit.register,
        and none of them avoid these problems. So, just make sure you
        call shutdown explicitly, in the process which created the
        NetworkDataChannel.
        
        The client processes don't seem to care, presumably because they
        are not running the server thread. Calling shutdown on the client
        does nothing.

### class OtherInputs
      Generic object to store any extra inputs and outputs used 
      inside the function being applied. This class was originally
      named for inputs, but in fact works just as well for outputs, 
      too. Any items stored on this will be persistent between 
      iterations of the block loop.
      
      When using multiple compute workers, copies of this object are given
      to each worker, where they can be modified as normal. A list of these
      copies is then given back on the :class:`rios.structures.ApplierReturn`
      object.

### class RasterizationMgr
      Manage rasterization of vector inputs, shared across multiple
      read workers within a single process. Not intended to be shared
      across compute workers on separate machines.

#### &nbsp;&nbsp;&nbsp;&nbsp; RasterizationMgr.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; RasterizationMgr.rasterize(self, vectorfile, rasterizeOptions, tmpfileMgr)
        Rasterize the given vector file, according to the given
        rasterizeOptions.
        
        Return the name of the temporary raster file.
        
        This is thread-safe. Any other thread trying to rasterize the
        same vector file will block until this has completed, and then
        be given exactly the same temporary raster file.

### class TempfileManager
      A single object which can keep track of all the temporary files
      created during a run. Shared between read worker threads, within
      a single process, so must be thread-safe. Not shared across processes.
      
      Includes methods to make and delete the temporary files.
      
      Constructor takes a single string for tempdir. All subsequent
      temp files will be created in a subdirectory underneath this.

#### &nbsp;&nbsp;&nbsp;&nbsp; TempfileManager.\_\_init\_\_(self, tempdir)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; TempfileManager.cleanup(self)
        Remove all the temp files created here

#### &nbsp;&nbsp;&nbsp;&nbsp; TempfileManager.mktempfile(self, prefix=None, suffix=None)
        Make a new tempfile, and return the full name

### class Timers
      Manage multiple named timers. See interval() method for example
      usage.
      
      Maintains a dictionary of pairs of start/finish times, before and
      after particular operations. These are grouped by operation names,
      and for each name, a list is accumulated of the pairs, for every
      time when this operation was carried out.
      
      The object is thread-safe, so multiple threads can accumulate to
      the same names. The object is also pickle-able.

#### &nbsp;&nbsp;&nbsp;&nbsp; Timers.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; Timers.formatReport(self, level=0)
        Format a simple report on the timing for the individual named
        timers. Passing level=0 produces a simple report with mean values
        for each named timer, level=1 adds some extra reporting summarizing
        the distribution of durations for each named timer.
        
        Return as a formatted string

#### &nbsp;&nbsp;&nbsp;&nbsp; Timers.getDurationsForName(self, intervalName)

#### &nbsp;&nbsp;&nbsp;&nbsp; Timers.interval(self, intervalName)
        Use as a context manager to time a particular named interval.
        
        Example::
        
            timings = Timers()
            with timings.interval('some_action'):
                # Code block required to perform the action
        
        After exit from the `with` statement, the timings object will have
        accumulated the start and end times around the code block. These
        will then contribute to the reporting of time intervals.

#### &nbsp;&nbsp;&nbsp;&nbsp; Timers.makeSummaryDict(self)
        Make some summary statistics, and return them in a dictionary

#### &nbsp;&nbsp;&nbsp;&nbsp; Timers.merge(self, other)
        Merge another Timers object into this one

### class WorkerErrorRecord
      Hold a record of an exception raised in a remote worker.

#### &nbsp;&nbsp;&nbsp;&nbsp; WorkerErrorRecord.\_\_init\_\_(self, exc, workerType, workerID=None)
        Initialize self.  See help(type(self)) for accurate signature.

## Values
    CW_AWSBATCH = 'CW_AWSBATCH'
    CW_ECS = 'CW_ECS'
    CW_NONE = 'CW_NONE'
    CW_PBS = 'CW_PBS'
    CW_SLURM = 'CW_SLURM'
    CW_SUBPROC = 'CW_SUBPROC'
    CW_THREADS = 'CW_THREADS'
