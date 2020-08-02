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
2. Provide access to the OSG Storage at the University of Chicago where users can store input files for jobs to the grid and 
output files (results) from their jobs
3. Provide an environment for the development of OSG appropriate workflows that will leverage distributed High ThroughPut 
computing. To facilitate such development a list of scientific software is accessible from the login node using `modules`. You can list availablle 
modules using the `module avail` command. You can load a module with the `module load <module_name>` command. More details on 
the module enviroment are discussed in [this section](#Data-access-from-OSG-grid).

## Storage access

Users will have access to the following storage locations on the Snowmass login node:

1. Home directory. As mentioned, your home directory will have 50GB of storage  available and can
be used for scripts, submission files and small size data. Home is network mounted on the login node and large input 
files for jobs on the grid should not be stored here.
2. Local storage in`/local-scratch`. *This is not available for user data at the moment. It will be*
*augmented with additional storage which would enable users to create work directories and submit jobs from there.*
*When it becomes available we will notify the users and update the documentation here.*
3. OSG storage (Ceph) accebible from the login node at `/mnt/ceph/osg/collab`. It is recommended that users store their 
data there in either of the two subdirectories:  
* For private user data: `/mnt/ceph/osg/collab/user/<user_id>`  
* For shared data among the members of the Snowmass21 collaboration:`/mnt/ceph/osg/collab/snowmass21`

Users can transfer data from external institutions to storage on Snowmass Connect using either of three following methods:
1. **scp**. For example: `scp -r <file_or_directory> <user_id>@login.snowmass.io:/mnt/ceph/osg/collab/user/<user_id>/.` will copy a file or a directory
from your local machine to your user directory on the OSG storage. The ssh-keys used for your profile on the Snowmass Connect portal 
must stored on the local machine.
2. **rsync**.
3. **Globus Connect** to transfer files to OSG storage only. A guide on how to gain access to the Globus door and instructions for transfering 
data to the OSG storage can be found here: [Globus Connect instructions](globus.md)

 
## Job submissions to the OSG

A typical HTCondor submission script is inlined below. 

    Universe = Vanilla
    Executable     = run.sh
    Requirements = && (HAS_MODULES =?= TRUE)
    Error   = output.err.$(Cluster)-$(Process)
    Output  = output.out.$(Cluster)-$(Process)
    Log     = output.log.$(Cluster)
    should_transfer_files = YES
    WhenToTransferOutput = ON_EXIT
    +ProjectName="snowmass21"
    Queue 1

The file run.sh is an executablle script that contains the list of commands that executes your workload 
along with any directives that move data as noted above. By default, the submission script above will use the HTCondor file transfer 
method to transfer the `Executable` to the remote host and the `Error`, `Output` and `Log` 
files back to user's directory on the submit host. Users can small files to and from the grid you can use the HTCondor 
file transfer method by including the following two lines in the submission 
script above:

    transfer_input_files = <comma separated files or directories>
    transfer_output_files = <comma separated files or directories>

Refer to the HTCondor manual for more information on customizing your submission scripts: https://research.cs.wisc.edu/htcondor/manual/v8.6/2_5Submitting_Job.html

## Data access from OSG grid 

As disussed above, users should place their science input data for processing on the Open Science Grid in /stash/collab/user/<user_id> or /stash/collab/project/snowmass21. There's no quota on this filesystem but expect about 10TB available. Data can be transferred to the grid as part of an OSG job using the stashcp tool. You can insert the following command in your execution script to move data from your collab space to the remote worker node where your 
job is running: 

    module load stashcache
    stashcp /osgconnect/collab/user/<user_id>/<input_file> .

To transfer data back to your collab space from the remote node that is running your job you can execute the following command:

    stashcp <output_file> stash:///osgconnect/collab/user/<user_id>/<output_file>

## Support and Consultation

Snowmass21 Connect is supported by the University of Chicago and Openscience Grid staff. To report issues with the service please submit a ticket to
support@opensciencegrid.org and request support from collaboration services. Consultation on submitting and running jobs at the OpenScience Grid
can be request by emailing: paschos@uchicago.edu
