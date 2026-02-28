# pyshepseg.cmdline.pyshepseg_segmentationworkercmd
    Main script for a segmentation worker running in a separate process.

## Functions
### def getCmdargs()
        Get command line arguments

### def mainCmd()
        Main entry point for command script. This is referenced by the install
        configuration to generate the actual command line main script.

### def popFromQue(que)
        Pop out the next item from the given Queue, returning None if
        the queue is empty.
        
        WARNING: don't use this if the queued items can be None

### def pyshepsegRemoteSegmentationWorker(workerID, host, port, authkey)
        The main routine to run a segmentation worker on a remote host.

