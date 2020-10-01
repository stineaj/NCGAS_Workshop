Compute Engine is the best place to start for understanding how to run computation jobs in GCP.  This allows you to start a virtual machine (VM) either with just a base Unix/Windows install or to lauch a machine with a disk image to include additional programs or configurations.

To creat a Compute Engine VM start at the GCP console
https://console.cloud.google.com/
click Compute Engine on the left menu.
On the GCP page for Compute Engine there is a quick start guide you can go through (takes about 10 minutes) for creating a VM. It is a useful refresher if it has been a while since you have created a VM instance and will cover many of the things we are going to talk about here.

Click the Create button in the center of the screen.

Select a name for the instance.  There are several items that are permanent for the instance and can't be changed later.  These are labelled in this screen as being permanent.

Select the Region that the instance will run in.  For this example we should use the menu to select us-east4 (Northern Virginia)  Regions determine the location of the data center running the compute instance.  When chosing a region to launch instances in, it can be important to know where any data you would be using is available.  The NCBI archive data is typically available to any US region.  Your own stored results or data may be limited in access to a smaller geographic area.  (We'll talk about this more when discussing buckets and storage.)

Select a Zone to run the instance in.  Zones are a further layer of localization.  There are regional resources that can only be accessed by other resources from within the same region as well as zonal resources that can only be accessed from within the same zone.  For our work we should be able to avoid issues with this by picking a zone and region we will work in and staying consistent with that for all our work.  More information about zones and regions can be found here.  https://cloud.google.com/compute/docs/regions-zones

We'll use a General-purpose E2 Series machine.  And select the e2-medium Machine type from the Shared core list.  This has access to 2 virtual cores and 4GB of memory.  Select CPU Platform Automatic
Choosing the correct size of CPU can have a big impact on the expense of compute costs.  For our examples we will be using low powered (and therefore lower cost per time) CPUs for learning.  For tasks that will take more resources like memory or are very compute intensive, a larger CPU may be necessary and may also be less expensive because it will be able to finish the task substantially faster.  Detailed pricing of the resources can be found here.  https://cloud.google.com/compute/all-pricing
At the time of writing this the machine selected above has a combined memory and vCPU cost of about $0.06 per hour.

In this example we'll start our instance with no image or container.  Select the 10GB persistent disk with Debian 10.

Also select the default service account and allow default access to the the VM.

Check the box to allow HTTP traffic to the instance.  This will open port 80 and give us access to connect to our instance more easily.

Click Create  This will begine the process of launching the VM instance.  You will be automatically taken to a page showing the running VM instances in GCP.  Once a green circle with a check inside of it appears next to your new instance, it is ready for you to connect to it.  

There are multiple ways to connect to a GCP instance.  You can use clinet like PuTTY or any ssh client you prefer to connect to the instance using HTTP access.  But you can also use a browser client from Google.  To connect this way find the Connect column menu for your instance and click the text SSH for your instance.  This will open a new window.  You may need to tell your browser to allow GCP to open new windows for your browser.

Once the window is open and finished connecting we will be presented with a command prompt for our compute instance.

This VM will have very few programs installed.  Frequent users of cloud services will typically go through the process of installing all the software they want on their instances and saving that as an image that can be launched from the Compute Engine.  Let's install the SRA Toolkit from NCBI to experience the process.

The current version of the SRA Toolkit is distributed from the NCBI FTP here.  ftp://ftp.ncbi.nlm.nih.gov/sra/sdk/current/
Our instance should have apt-get installed.  We can verify this by typing 'which apt-get' in the console and making sure a location is returned when we hit enter.  

This means the install script we will want to run for the SRA Toolkit is here: ftp://ftp.ncbi.nlm.nih.gov/sra/sdk/current/setup-apt.sh
We'll use curl to copy that script to our VM instance with the following command.
curl -O ftp://ftp.ncbi.nlm.nih.gov/sra/sdk/current/setup-apt.sh

Next we need to make the program into an executable file with
chmod +x setup-apt.sh

Now we'll run the install script using sudo so that it has suffcient permissions to install necessary libraries and packages.
sudo setup-apt.sh

This will take a few minutes to install all the required components.  The last message in the install process with tell us to source a shell script that was installed with the toolkit.  This will add the newly installed toolkit programs to our path.
source /etc/profile.d/sra-tools.sh

The final step to getting the toolkit installed will be to run vdb-config program to configure our toolkit install.
vdb-config --interactive

This will open a text based graphical user interface (GUI) to set some required parameters.  For our use the important we need to do is accept charges and report our cloud machine information so that the toolkit can make decisions about where is best to access sequence data in SRA from.  We will do this by typing the letter highlighted in red on the screen to move between menus and set options.

Type g in the window to select the GCP menu.

Type e to accept charges for GCP so that we as the users agree to pay costs for egress of data to different storage regions or access to user owned buckets.  In this seminar we do not expect to generate any charges from this type of usage.

Type r to repor the cloud instance identity to the toolkit.  The most important information provided here is the cloud provider and region.  Reporting this information to the toolkit allows it to access data in the same cloud region we are running in when possible.

Type s to save the configuration.  Type o to confirm.

Type x to exit the configuration.  We have finished configuring the SRA Toolkit for accessing data on a GCP VM.

To test our configuration, lets try accessing a sequencing data run from SRA.
prefetch SRR000001

We should see several lines of information appear on the screen with the last line telling us we have downloaded the data successfully.  
1) 'SRR000001' was downloaded successfully

If we list the directory SRR000001 we will see the downloaded data file in the directory.
ls SRR000001

We can also use the SRA Toolkit to convert the data we downloaded into other formats, like fastq.  We will use the fasterq-dump program to produce fastq data.  This program has many options but if we run it with the default choices it will output a fastq file for each read in a paired data set.  And if there are unmated reads in the run, it will output a third file with just the unmated reads.
fasterq-dump SRR000001

Note that we don't need to use the specific file (SRR000001/SRA000001.sra), only the accession since the toolkit is able to locate the data we have downloaded by using only the accession.  If we look at the fastq files that were produced from this we'll see three files.
SRR000001.fastq  SRR000001_1.fastq  SRR000001_2.fastq

The first file is the unmated reads while the files with the _1 and _2 are the mated reads. 

To review we have started an new VM instance, installed the SRA Toolkit and the additional libraries it needed, downloaded a run from the SRA, and converted that run into fastq format.  But this particular VM has accomplished all we need to do with it right now.  For later work we will be starting our VM from an image that has more software installed for us.  To close the terminal window type
exit

This should close the browser window for our instance.  Our VM is still running however.  We need to go back to our compute instances in the console.  Our instances can be found by coing to the Compute Engine page of the console and selecing VM instances from the menu on the left of the screen.

This is the page we previously launched our client window from to access the VM.  We can perform several important actions by clicking the three dots to the right of the Connect column in the table showing our instances.  We can stop or suspend instances that we will want to return to later but are not using currently.  Or we can delete instances that we do not plan to use again.

Click Delete to stop and delete our instance and remove the data we downloaded.  
More information about the various states an instance can be found here.  https://cloud.google.com/compute/docs/instances/instance-life-cycle



