# rios.cmdline.rios_computeworker
    Main script for a compute worker running in a separate process.

## Functions
### def getCmdargs()
        Get command line arguments

### def mainCmd()
        Main entry point for command script. This is referenced by the install
        configuration to generate the actual command line main script.

### def riosRemoteComputeWorker(workerID, host, port, authkey)
        The main routine to run a compute worker on a remote host.

