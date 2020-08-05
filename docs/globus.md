# Transferring Data to OSG Storage

Data used as input for jobs to the OSG grid should be transferred to the OSG storage allocated for the Snowmass21 project
using Globus Online. Instructions on how to set up Globus Connect Personal can be found 
[here](https://www.globus.org/globus-connect-personal). Access to the OSG storage endpoint is enabled by authenticating 
against the Globus collection "OSG Connect CI Logon" using the GLobus Connect client. 
You can search for the collection by name in the search bar of the File Manager.

In order to access the OSG storage for Snowmass via Globus online, users must have an institutional 
based grid certificate issued by CILogon. To obtain one follow the steps below:

1. Logon with your institutional credentials at [http://cilogon.org](http://cilogon.org)
2. Select "Create a Password Protected Certificate". Enter a password and download your encrypted certificate, named usercred.p12. 
The certificate can be obtained 
by using the openssl pcks12 command as: `openssl pkcs12 -in [your-cert-file] -clcerts -nokeys -out usercert.pem`
3. Email to [paschos@uchicago.edu](paschos@uchicago.edu) the output of the following command which will print out your 
DN (Distinguish Name): `openssl x509 -in usercert.pem -noout -subject`

Once your DN has been entered in the user access list you will be able to access the OSG Connect CI Logon collection with the Globus Connect client by 
validating with your institution credentials. Navigate to the OSG Snowmass21 Collaborations Connect storage by typing in the Path box `/stash/collab`. You can then navigate to your user directory as shown in the example below:

![](snowmass_3.png)

Shown in the image above are two possible destinations for the data.

1. Navigate to `/stash/collab/project/snowmass21` if data are to be shared by multiple users.
2. Navigate to `/stash/collab/user/<user_id>` if data are for the exclusive use of a single user.
In both cases, users can create subdirectories and organize content by either using the Globus client interface or from the login.snowmass21.io node.

On the right panel of the Globus Connect client tool you can search and connect to another collection. 
The latter can be your own laptop/server or a collaboration end point that has provided a Globus Connect door for the researchers to use. To transfer files 
you can select the list files from your local computer and then select Start. To transfer files out simply reverse the direction of the process.

 **Important** Users can not access their home directories in the snowmass21 login node over the Globus door. However, users have access to the /stash/collab directory when they login to login.snowmass21.io. 
 Files can be moved or copied over to their home directory. 
