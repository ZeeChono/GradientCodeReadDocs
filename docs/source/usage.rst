Usage
=====

.. _installation:

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
- Save the .pem key on your machine under **~/.ssh** 

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
- Storage Configuration, for free tier, you have 30 gb in total, free to modify it as you need

   .. image:: assets/7_instance.png
      :alt: Set the storage option
      :width: 600px
      :height: 350px
      :align: center
- Under the summary, input the number of instances you would like to instantiate, then click "Launch instance"
- Goto instances dashboard and check their status, public ipv4 address, and then add a config file under the same folder


Step 4: Test ssh from Local to your Instances
---------------------------------------------
If you have configured your **~/.ssh/config** file and your key file correctly, open your terminal and type

   .. code-block:: 

      ssh master

   .. image:: assets/ssh_configfile.png
      :alt: SSH config file settings
      :width: 600px
      :height: 350px
      :align: center

Please make sure the instance you want to connect to is online and the public ipv4 address is up to date in the config file (Because AWS allocates dynamic ipv4 to free tier users).


Step 5: Setting Up Dev Environment of Each Instance
---------------------------------------------------
This step is pretty much copy, paste and wait. By default, the AMI chosen at step 2 would have python3 installed for you. So we will only install the following packages for our project:

   .. code-block:: 

      sudo apt install python3-mpi4py
      sudo apt install python3-numpy
      sudo apt install python3-pandas
      sudo apt install python3-sklearn

   You could copy them line by line and wait for all to be installed.

Notice that this step is expected to be completed for **each of your instance**.


Step 6: Test MPI Run for Each Instance
--------------------------------------
Create a hello_world.py python file for testing purpose.

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

If no error is reported, this should ensure that you have all the required packages ready for the project.


Step 7: Enable the Communication Among Instances
------------------------------------------------
Similarly to what we have done in step 2, we are going to setup **~/.ssh** for each instance. We need to copy our .pem key file to each instance by calling the following command on your local PC:

   .. code-block:: 

      scp ~/.ssh/<your key>.pem master:~/.ssh
      scp ~/.ssh/<your key>.pem w1:~/.ssh
      scp ~/.ssh/<your key>.pem w2:~/.ssh

   If you have more worker nodes, please go on

Then ssh into your instances, and create a similar config file under the .ssh folder, here is an example of my master node .ssh/config file:

   .. code-block:: 

      Host w1
        HostName ip-172-31-15-69.ca-central-1.compute.internal   # this should be the private ipv4 or dns name of your instance, which can be found on dashboard
        User ubuntu
        IdentityFile  ~/.ssh/first_instance.pem

      Host w2
        HostName ip-172-31-13-65.ca-central-1.compute.internal
        User ubuntu
        IdentityFile ~/.ssh/first_instance.pem

Secondly, we will have to modify the file **/etc/hosts**. Here's an example of how my file looks like on the master node. Update yours accordingly for each instance.

   .. code-block:: 

      127.0.0.1 localhost
      172.31.15.69 w1         # input ipv4 private and the same name in the .ssh hostname
      172.31.13.65 w2
      
      # The following lines are desirable for IPv6 capable hosts
      ::1 ip6-localhost ip6-loopback
      fe00::0 ip6-localnet
      ff00::0 ip6-mcastprefix
      ff02::1 ip6-allnodes
      ff02::2 ip6-allrouters
      ff02::3 ip6-allhosts

Besides this, we should also enable TCP connection between these instances under the same **Network Security Group**. On dash board of the EC2, click Security Groups on the sidebar, this will bring you to the group we have just created in the step 2. Click "Edit inbound rules" after you enter the security group you created, add a new TCP rule as shown in the following screenshot.


After we have finished all above cluster settings, try manually ssh to each other before mpirun command to ensure that the communication in cluster is setup correctly. 


Step 8: Final Cluster MPI Test
------------------------------
Now open your ssh master terminal, 

   .. code-block:: 

      mpirun -np 3 -H localhost,w1,w2 python3 hello_world.py

If again no errors get reported, congratulations! Your cluster is now ready to go.
