# Snowmass Connect

The purpose of this documentation is to facilate the onboarding of researchers with the Snowmass21 collaboration in accessing
the Open Science Grid (OSG) via the Snowmass Connect service hosted at the University of Chicago.
  
## Getting Started

Snowmass Connect is a job submission service providing access to the distributed cyberinfrastructure of the Open Science Grid. To sign up, visit [https://connect.snowmass21.io](https://connect.snowmass21.io). It is important that you upload your ssh-keys following the instructions on the portal. The process will create your
home directory on the Snowmass submit node and grant you access via passwordless ssh.

## Login to the submit node

Once you upload your ssh-keys in the Snowmass Connect portal you can connect to the login node via passwordless ssh as:

`ssh <user_id>@login.snowmass21.io` 

You can find your `<user_id>` from your profile on the Snowmass Connect portal. 
The login node is also a submission node for jobs to the Open Science Grid. Upon login you will land in your home directory `/home/user_id`. Your home 
directory has 50GB of quota. Use your home directory to store job submission files, scripts and code. Do **not** store large files (larger than 300 MB) in your home directory for the purpose of submissions to the OSG. 

The Snowmass login node provides a service to the users of the collaboration in the following ways:

1. Provide users a gateway to the Open Science Grid in order to run their production computational workflows via job submissions to the OSG HTCondor pool
2. Provide access to the OSG Storage at the University of Chicago where users can stage input files for grid jobs and collect
output from their jobs
3. Provide an environment for the development of OSG appropriate workflows that will leverage distributed High ThroughPut 
computing. To facilitate such development a list of scientific software is accessible from the login node using `modules`. You can list availablle 
modules using the `module avail` command. You can load a module with the `module load <module_name>` command. Users can request remote worker
nodes where the module environment is available, discussed in the [Job submission](#Job-submissions-to-the-OSG) section.


## Storage access

This section describes the storage locations accessible by the users on the on when connected to Snowmass login node. It also describes the means by which
users can move data in from their home institutions.

1. Home directory. As mentioned above, your home directory will have 50GB of storage  available and can
be used for scripts, submission files and small size data. Large input files for jobs on the grid should not be stored here.
2. OSG storage accebible from the login node at `/collab`. Users will receive a 1TB quota for their personal user space, and there is a shared 50TB quota for the project space. Larger space allocations can be accommodated on a case-by-case basis. Users should store their data in either of two subdirectories:  
    * For private user data: `/collab/user/<user_id>`  
    * For shared data among the members of the Snowmass21 collaboration:`/collab/snowmass21`
3. Local storage in`/scratch`. *This is not available for user data at the moment. 200 TB will be*
*provisioned to allow users to create work directories and launch jobs from there.*
*Users will be notified when this storage becomes available. Documentation here will be updated accordingly with more details.

Users can transfer data from external institutions to Snowmass Connect storage using any of the three following methods:

1. **scp**. For example: `scp -r <file_or_directory> <user_id>@login.snowmass.io:/collab/user/<user_id>/.` will copy a file or a directory
from your local machine to your user directory on the OSG storage. The ssh-keys used for your profile on the Snowmass Connect portal 
must stored on the local machine.

2. **rsync**. For example: `rsync -avz -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" --progress test.transfer <user_id>@login.snowmass21.io:dump/` will copy the `test.transfer` file in the `/home/<user_id>/dump/` directory. If the directory 
does not exist, it will be created. As in `scp` the ssh-keys used for your profile on the Snowmass Connect portal 
must stored on the source machine.

3. **Globus Connect** to transfer files to OSG storage only. A guide on how to gain access to the Globus door and instructions for transfering 
data to the OSG storage can be found here: [Globus Connect instructions](globus.md)

 
## Job submissions to the OSG

This section provides a short introduction to submitting jobs to the OSG from the Snowmass Connect login node. 
A minimal HTCondor submission script, `myjob.submit`, to the OSG is inlined below:

    Universe = Vanilla
    Executable     = run.sh
    Error   = output.err.$(Cluster)-$(Process)
    Output  = output.out.$(Cluster)-$(Process)
    Log     = output.log.$(Cluster)
    should_transfer_files = YES
    WhenToTransferOutput = ON_EXIT
    request_cpus = 1
    request_memory = 1 GB
    +ProjectName="snowmass21"
    Queue 1

Refer to the [HTCondor manual](https://htcondor.readthedocs.io/en/stable/users-manual/index.html) for more information on the declared parameters and on customizing your submission scripts.

When the HTCondor script above is submitted, the user would request a remote worker node with 1 core and 1 GB to run the `run.sh` executable. In this case, `run.sh` is a shell script that contains a list of commands that executes your workload on the worker node.  For example: 

    #/bin/bash
    ./code_executable <input_file> <output_file>
    <additional commands>

The parameter `should_transfer_files = YES` instructs HTCondor to use the HTCondor file transfer 
method to transfer the `Executable` to the remote host and the job files `Error` (stderr) , `Output` (stdout) and `Log` 
back to user's directory on the submit host. Users have a number of options to transfer
their code executables and input/output files to the remote worker node, described in the next section.

Users can submit the job script to the OSG via the HTCondor command on the Snowmass login node: 
`condor_submit myjob.submit`, which will return a unique `<JobID>` number. 
You can use the `<JobID>` to query the status of your job with `condor_q <JobID>`

For an introduction on managing your jobs with HTCondor we refer to [this](https://opensciencegrid.org/user-school-2019/#materials/day1/files/osgus19-day1-part1-intro-to-htc.pdf) presentation by the OSG:

###  Job Submission Guidelines

1. If your application/code was built or depends on modules used on the snowmass21 login node and it dynamically links against libraries of the module environment you would need to ensure that these modules are also availablle and loaded on the remote worker node. To do so:
    * Insert the following parameter in your submission script: `Requirements = (HAS_MODULES =?= TRUE)`. This will request a worker node on a site where the OSG modules are available
    * Before you invoke your executable inside the `run.sh` script load the modules as: `module load module1 module2`
  
2. You must always declare your project name, `+ProjectName="snowmass21"`, in your HTCondor submission script to:
    * Ensure your job is validated for HTCondor to run it on the OSG grid
    * Job statistics are properly collected and displayed on the OSG monitoring dashboard for the snowmass project: `https://gracc.opensciencegrid.org/`

## Data Management and Grid Transfer

This section describes recommendations and options for transferring data to the from remote woker nodes as part of a job submission to the OSG.
As disussed above, users should place their input data for processing on the Open Science Grid in `/collab/user/<user_id>` or `/collab/project/snowmass21`. Data can be transferred to the grid as part of an OSG job using three different methods depending on the file size.

1. HTCondor File Transfer. This method is recommended for the majority of computational workflows running on the OSG. Users can employ this method if the total size of the input data per job does not exceed 1 GB. In addition, OSG recommends that the output data per job that need to be transfered back does not exceed 1 GB as well. To enable HTCondor File transfers for your input and output data insert the following parameters anywhere in your HTCondor submit file:

        transfer_input_files = <comma separated files or directories>
        transfer_output_files = <comma separated files or directories>

2. OSG's StashCache. This method is recommended for input files larger than 1 GB each or 10 GB total from all input data. The recommended upper limit for the output files to be transfered back from the remote node is 10 GB per job. Users can use the stashcp tool to transfer data from their `/collab` space only to the remote host. You can insert the following command in your execution script to transfer data from `/collab/user/<user_id>` to the local
directory on the remote worker node where your job is running: 

        module load stashcache
        stashcp /osgconnect/collab/user/<user_id>/<input_file> .
To transfer data back to your collab space from the remote node run the following command in your execution script:

        stashcp <output_file> stash:///osgconnect/collab/user/<user_id>/<output_file>
        
3. If the filesize of each input dataset exceeds 10 GB then an alternative method for transfers is the GridFTP protocol using the gfal-copy tool. Please reach out 
for a consultation to discuss if your workflow can benefit from access to a GridFTP door.

## Job Examples to the OSG

A list of HTCondor submission examples to the OSG with instructions for users to practice with is included here: [List of Job Examples](examples.md)

## Support and Consultation

Snowmass21 Connect is supported by the University of Chicago and Open Science Grid staff. To report issues with the service please submit a ticket to
[OSG support](support@opensciencegrid.org) and request support from Collaboration Services. Consultation on submitting and running jobs at the OpenScience Grid
can be requested by emailing: [paschos@uchicago.edu](paschos@uchicago.edu)
