# Overview


## Getting Started

Snowmass Connect is a job submission service providing access to the Open Science Grid. To sign up, visit [https://connect.snowmass21.io/](https://connect.snowmass21.io).

## Login to the submit node

After you have uploaded your ssh-keys in the Snowmass Connect portal you can connect to the login node as:

`ssh login.snowmass21.io` 

The login node is also a submission node for jobs to the Open Science Grid. Upon login you will in your home directory that has 50GB of quota. 
Use your home directory to store submission files and scripts. 

## Data placement

Users should place their input data in /collab/user/<user_id>. There's no quota on this filesystem but expect about 10TB available. 
You can move data to the grid as part of an OSG job submission using the stashcp tool. 
You can insert the following command in your execution script to move data from your collab space to the remote worker node where your 
job is running: 

`module load stashcache`

`stashcp /osgconnect/collab/user/<user_id>/<input_file> .`

To transfer data back to your collab space from the remote node that is running your job you can execute the following command:

`stashcp <output_file> stash:///osgconnect/collab/user/<user_id>/<output_file>`


## Job submissions

A typical submission script is inlined below. 

`Universe = Vanilla`

`Executable     = run.sh`

`Requirements = && (HAS_MODULES =?= TRUE)`

`Error   = output.err.$(Cluster)-$(Process)`

`Output  = output.out.$(Cluster)-$(Process)`

`Log     = output.log.$(Cluster)`

`WhenToTransferOutput = ON_EXIT`

`+ProjectName="snowmass21"`

`Queue 1`

The file run.sh is a shell executablle script that contains your execution commands along with any directives to move the data as noted above. 
For small files to and from the grid you can use the HTCondor file transfer method by including the following two lines in the submission 
script above:

`transfer_input_files = <comma separated files>`

`should_transfer_files = YES`

Refer to the HTCondor manual for more information on customizing your submission scripts: https://research.cs.wisc.edu/htcondor/manual/v8.6/2_5Submitting_Job.html



