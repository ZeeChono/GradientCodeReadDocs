Cluster Setup
=============


Step 1: Creating the AWS Root Account
-------------------------------------

To setup the customized cluster, we shall first create an AWS account. 
Goto https://portal.aws.amazon.com/billing/signup#/start/email to register for a new account step by step.


Step 2: Search for EC2
----------------------

Once finishing the new account sign-up, you will be redirected to the homepage of the AWS console. On the top, enter "EC2" in the search bar and click the first result.  

After being brought to the EC2 dashboard, click launch instances in the middle of the screen.

   .. image:: assets/1_EC2_search.png
      :alt: EC2 search
      :width: 600px
      :height: 350px
      :align: center
   .. image:: assets/2_launch_instance.png
      :alt: Launch a new instance
      :width: 600px
      :height: 350px
      :align: center



Step 3: Configure and Launch Instances
--------------------------------------

- Input the name (editable afterwards) and select the correct AMI (Amazon Machine Image)

   .. image:: assets/3_select_IMI.png
      :alt: Select correct IMI
      :width: 600px
      :height: 350px
      :align: center
- Select the instance type -- t2.micro for our project specifically
- Create a new key pair for ssh (you can reuse the same key for as many instances as you want)

   .. image:: assets/4_instance.png
      :alt: Choose instance type and ssh key pair
      :width: 600px
      :height: 350px
      :align: center
- Save the .pem key on your local machine under **~/.ssh** 

   .. image:: assets/5_instance.png
      :alt: Save ssh key pair
      :width: 400px
      :height: 400px
      :align: center
- Edit the Network Settings as follows, give it a name for later simplicity to search it up

   .. image:: assets/6_instance.png
      :alt: Network settings
      :width: 300px
      :height: 400px
      :align: center
- Storage Configuration, for free tier, you have 30 gb in total, free to modify it as you need. (But it may have the least storage requirement)

   .. image:: assets/7_instance.png
      :alt: Set the storage option
      :width: 600px
      :height: 350px
      :align: center
- Under the summary, input the number of instances you would like to instantiate, then click "Launch instance"
- Goto instances dashboard and you can check their status, public ipv4 address, CPU utilization etc.

   .. image:: assets/EC2_dashboard.png
      :alt: Set the storage option
      :width: 400px
      :height: 400px
      :align: center


Step 4: Test ssh from Local to your Instances
---------------------------------------------
First, let us configure the **~/.ssh/config** file with the .pem key we have downloaded in the last step. The host names here in the screenshot are the corresponding ipv4 address which can be found on the dashboard of EC2. (The AWS servers use dynamic public ipv4, you can pay to use static ipv4 as well)

   .. image:: assets/ssh_configfile.png
      :alt: SSH config file settings
      :width: 600px
      :height: 350px
      :align: center

If you have configured your **~/.ssh/config** file and your key file correctly, open your terminal and type

   .. code-block:: 

      ssh master

   

Please make sure the instance you want to connect to is online and the public ipv4 address is up to date in the config file (Because AWS allocates dynamic ipv4 to free tier users).


Step 5: Setting Up Dev Environment of Each Instance
---------------------------------------------------
This step is pretty much copy, paste and wait. By default, the AMI chosen at step 2 would have python3 installed for you. So we will only install the following packages for our project:

   .. code-block:: 

      sudo apt-get update
      sudo apt install make
      sudo apt install python3-mpi4py
      sudo apt install python3-numpy
      sudo apt install python3-pandas
      sudo apt install python3-sklearn

   You could copy them line by line and wait for all to be installed.

Notice that this step is expected to be completed for **each of your instance**.


Step 6: Test MPI Run for Each Instance
--------------------------------------
Create a **hello_world.py** python file for testing purpose.

   .. code-block:: 

      from mpi4py import MPI
      import numpy as np
      import pandas as pd
      import sklearn
      
      comm = MPI.COMM_WORLD
      rank = comm.Get_rank()
      size = comm.Get_size()
      
      # Get the name of the processor
      processor_name = MPI.Get_processor_name()
      
      # Print the rank, size, and processor name
      print(f"Hello from rank {rank} out of {size} processors on host {processor_name}")

Then in the terminal run:

   .. code-block:: 

      mpirun -np 1 python3 hello_world.py

If no error is reported, you have all the required packages ready for the project.


Step 7: Enable the Communication Among Instances
------------------------------------------------
Similarly to what we have done in step 2, we are going to setup **~/.ssh** for the master instance. We need to copy our .pem key file to the master instance by calling the following command on your local PC:

   .. code-block:: 

      scp [path to your key].pem master:~/.ssh

Then ssh into your master node, and create a similar config file under the .ssh folder, here is an example of my master node .ssh/config file:

   .. code-block:: 

      Host w1
        HostName 172.31.15.69   # this should be the private ipv4 or dns name of your instance, which can be found on dashboard
        User ubuntu
        IdentityFile  ~/.ssh/first_instance.pem

      Host w2
        HostName 172.31.13.65
        User ubuntu
        IdentityFile ~/.ssh/first_instance.pem

Besides this, we should also enable TCP connection between these instances under the same **Network Security Group**. On dash board of the EC2, click Security Groups on the sidebar, this will bring you to the group we have just created in the step 2. Click "Edit inbound rules" after you enter the security group you created, add a new TCP rule as shown in the following screenshot.

   .. image:: assets/8_network_setting1.png
      :alt: Inbound rule setting1
      :width: 600px
      :height: 300px
      :align: center   
   .. image:: assets/9_network_setting2.png
      :alt: Inbound rule setting2
      :width: 600px
      :height: 300px
      :align: center 
   .. image:: assets/10_network_setting3.png
      :alt: Inbound rule setting3
      :width: 600px
      :height: 300px
      :align: center 

**Notice that the source of your newly added Custom TCP rule should be constrained within your secutrity group, otherwise you are opening your all ports to the public internet which is dangerous.**

After we have finished all the cluster settings, try manually ssh to each other node on each node, before mpirun command to ensure that the communication in the cluster is set up correctly. 


**Note**: In case of the permission of bad .pem key error reported on the AWS server, try the following command:

   .. code-block:: 

      chmod 600 ~/.ssh/<your key name>.pem    # this modify the ssh key to read only



Step 8: Final Cluster MPI Test
------------------------------
Now open your ssh master terminal, 

   .. code-block:: 

      mpirun -np 3 -H localhost,w1,w2 python3 hello_world.py

If again no errors get reported, congratulations! Your cluster is now ready to go.
