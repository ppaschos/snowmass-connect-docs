# Snowmass Connect

The purpose of this documentation is to facilate the onboarding of researchers with the Snowmass21 collaboration in accessing
the Open Science Grid (OSG) via the Snowmass Connect service hosted at the University of Chicago.
  
## Getting Started

Snowmass Connect is a job submission service providing access to the Open Science Grid. To sign up, visit [https://connect.snowmass21.io](https://connect.snowmass21.io). It is important that you upload your ssh-keys following the instructions on the portal. The process will create your
home directory on the Snowmass submit node and grant you access via passwordless ssh.

## Login to the submit node

Once you upload your ssh-keys in the Snowmass Connect portal you can connect to the login node via passwordless ssh as:

`ssh <user_id>@login.snowmass21.io` 

You can find your `<user_id>` from your profile on the Snowmass Connect portal. 
The login node is also a submission node for jobs to the Open Science Grid. Upon login you will land in your home directory `/home/user_id`. Your home 
directory has 50GB of quota. Use your home directory to store job submission files, scripts and code. Do **not** store large files (larger than 300 MB) in your home directory for the purpose of submissions to the OSG. 

The Snowmass login node provides a service to the users of the collaboration in the following ways:

1. Provide users a gateway to the Open Science Grid in order to 
run their production computational workflows via job submissions to the OSG HTCondor pool
2. Provide access to the OSG Storage at the University of Chicago where users can stage input files for grid jobs and collect
output from their jobs
3. Provide an environment for the development of OSG appropriate workflows that will leverage distributed High ThroughPut 
computing. To facilitate such development a list of scientific software is accessible from the login node using `modules`. You can list availablle 
modules using the `module avail` command. You can load a module with the `module load <module_name>` command. Users can request remote worker
nodes where the module environment is available, discussed in the [Job submission](#Job-submissions-to-the-OSG) section.

## Storage access

Users will have access to the following storage locations when connected to the Snowmass login node:

1. Home directory. As mentioned above, your home directory will have 50GB of storage  available and can
be used for scripts, submission files and small size data. Home is network mounted on the login node and large input 
files for jobs on the grid should not be stored here.
2. Local storage in`/local-scratch`. *This is not available for user data at the moment. Additional storage will be*
*provisioned which would enable users to create work directories and submit jobs from there.*
*When it becomes available we will notify the users and update the documentation here.*
3. OSG storage (Ceph) accebible from the login node at `/collab`. Users should store their 
data in either of two subdirectories:  
* For private user data: `/collab/user/<user_id>`  
* For shared data among the members of the Snowmass21 collaboration:`/collab/snowmass21`

Users can transfer data from external institutions to Snowmass Connect storage using any of the three following methods:
1. **scp**. For example: `scp -r <file_or_directory> <user_id>@login.snowmass.io:/collab/user/<user_id>/.` will copy a file or a directory
from your local machine to your user directory on the OSG storage. The ssh-keys used for your profile on the Snowmass Connect portal 
must stored on the local machine.
2. **rsync**.
3. **Globus Connect** to transfer files to OSG storage only. A guide on how to gain access to the Globus door and instructions for transfering 
data to the OSG storage can be found here: [Globus Connect instructions](globus.md)

 
## Job submissions to the OSG

A minimal HTCondor submission script, `myjob.submit`, to the OSG is inlined below. 

    Universe = Vanilla
    Executable     = run.sh
    Error   = output.err.$(Cluster)-$(Process)
    Output  = output.out.$(Cluster)-$(Process)
    Log     = output.log.$(Cluster)
    should_transfer_files = YES
    WhenToTransferOutput = ON_EXIT
    +ProjectName="snowmass21"
    Queue 1

Refer to the HTCondor manual for more information on the declared parameters and on customizing your submission scripts: https://htcondor.readthedocs.io/en/stable/users-manual/index.html

When the script above is submitted, the user would request a remote worker node 
to run the `run.sh` executable. In this case, `run.sh` is a shell script that contains a list of commands 
that executes your workload on the worker load.  For example: 

    #/bin/bash
    ./code_executable <input_file> <output_file>
    <additional commands>

The parameter `should_transfer_files = YES` instructs Condor to use the HTCondor file transfer 
method to transfer the `Executable` to the remote host and the job files `Error` (stderr) , `Output` (stdout) and `Log` 
back to user's directory on the submit host. Users have a number of options to transfer
their code executables and input/output files to the remote worker node and is described in the next section.

Users can submit the job script to the OSG via the condor command on the Snowmass login node: 
`condor_submit myjob.submit`, which will return a unique `<JobID>` number. 
You can use the `<JobID>` to query the status of your job with `condor_q <JobID>`

For an introduction on managing your jobs with condor we refer to this presentation by the OSG
https://opensciencegrid.org/user-school-2019/#materials/day1/files/osgus19-day1-part1-intro-to-htc.pdf

### Notable points

1. If your application/code was build or depends on modules used on the snowmass21 login node you must 
ensure that these modules are loaded also on the remote worker node. To do so:
* Insert the following parameter in your submission script: `Requirements = (HAS_MODULES =?= TRUE)`. This will 
request a worker node on a site where the OSG modules are available.
* Load the modules in the executable script, `run.sh` before you invoke your executable as: `module load module1 module2 ...`


## Data Management and Grid Transfers

As disussed above, users should place their input data for processing on the Open Science Grid in `/collab/user/<user_id>` or `/collab/project/snowmass21`. There's no quota on this filesystem but expect about 10TB available. Data can be transferred to the grid as part of an OSG job using four different methods depending on the file size.

1. HTCondor File Transfer for files less than 100 MB.

    transfer_input_files = <comma separated files or directories>
    transfer_output_files = <comma separated files or directories>
  
2. Unix tools for datasets less than 1 GB such as rsync can be invoked from your execution script 
on the remote host to transfer files from `/collab` by connecting to the submit host.
3. OSG's StashCache for files greater than 1 GB. Users can use the stashcp tool to transfer data in their `/collab` space to the remote host. 
You can insert the following command in your execution script to  move data from `/collab/user/<user_id>` to the local
directory on remote worker node where your job is running: 

    module load stashcache
    stashcp /osgconnect/collab/user/<user_id>/<input_file> .

To transfer data back to your collab space from the remote node that is running your job you can execute the following command:

    stashcp <output_file> stash:///osgconnect/collab/user/<user_id>/<output_file>
4. If the filesize each dataset exceeds 2 GB an alternative method for transfers is the GridFTP protocol using the gfal-copy tool. Please reach out 
for a consultation to discuss if your workflow can benefit from access to a GridFTP door. 

## Support and Consultation

Snowmass21 Connect is supported by the University of Chicago and Openscience Grid staff. To report issues with the service please submit a ticket to
support@opensciencegrid.org and request support from collaboration services. Consultation on submitting and running jobs at the OpenScience Grid
can be request by emailing: paschos@uchicago.edu
