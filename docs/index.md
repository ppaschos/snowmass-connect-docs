# Snowmass Connect

The purpose of this documentation is to provide critical information for researchers with the Snowmass21 collaboration to access
the Open Science Grid (OSG) via the Snowmass Connect service hosted at the University of Chicago.

## Getting Started

Snowmass Connect is a job submission service providing access to the Open Science Grid. To sign up, visit [https://connect.snowmass21.io](https://connect.snowmass21.io). It is important to upload your ssh-keys following the instructions on the site. The process will create your
home directory on the Snowmass submit node and grant you access via passwordless ssh.

## Login to the submit node

Once you upload your ssh-keys in the Snowmass Connect portal you can connect to the login node via passwordless ssh as:

`ssh <user_id>@login.snowmass21.io` 

You can find your `<user_id>` from your profile on the Snowmass Connect portal. 
The login node is also a submission node for jobs to the Open Science Grid. Upon login you will land in your home directory `/home/user_id`. Your home 
directory has 50GB of quota. Use your home directory to store job submission files, scripts and code. Do **not** store large files (larger than 300 MB) in your home directory for the purpose of submissions to the OSG. 

The Snowmass login node provides a service to the users of the collaboration in the following ways:

1. Provide users a gateway to the OpenScience Grid in order to 
run their production computational workflows via job submissions to the OSG HTCondor pool
2. Provide access to the OSG Storage at the University of Chicago where users can store input files for jobs to the grid and 
output files (results) from their jobs
3. Provide an environment for the development of OSG appropriate workflows that will leverage distributed High ThroughPut 
computing. To facilitate such development a list of scientific software is accessible from the login node using `Modules`. You can list availablle 
module using the `module avail` command. You can load a module with the `module load <module_name>` command. More details on 
the module enviroment are discussed in [this section](#Data-access-from-OSG-grid).

## Storage access

[Globus Connect instructions](globus.md)

 
## OSG Job submissions

A typical submission script is inlined below. 

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

## Dataaccess from OSG grid 

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
