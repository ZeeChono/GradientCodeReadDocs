Run Simulation
==============


Step 1: Project Clone
---------------------

Firstly, we need to clone the project code into the cluster master node. After you ssh into the master server, run the following command:

.. code-block:: 

    git clone https://github.com/ZeeChono/gradient_coding_mds


Step 2: Project Configuration
-----------------------------

To make one's life easier, this project directory setup should follow as is shown in the screenshot below::

    ~/
    ├── cluster_dir_setup.sh    # the script to scp running scripts and distribute data partitions to each worker; TODO: can be found inside the gradient_coding_mds/
    ├── copy.sh                 # the script that called by the cluster_dir_setup.sh
    ├── dataset                 # entry point of dataset
    │   └── amazon-dataset
    |       ├── test.csv        # TODO: these csv files needs to be scp to the master node mannually
    |       └── train.csv
    ├── gradient_coding_mds/    # TODO: cloned project directory
    │   ├── main.py
    │   └── ...
    └── hello_world.py          # the script used to test MPI setup which appears in the last section :ref:`cluster_setup`



This project relies on the Makefile to build the project and start the simulation. There are several important parameters that the user 
should understand::

    # No. of workers which includes the master node
    N_PROCS=11
    
    # No. of stragglers you assumed for the simulation, usually useful for the exact gradient recovery schemes.
    N_STRAGGLERS=1
    
    # For partially coded version: how many pieces of workload will one worker be handling.
    N_PARTITIONS=10
    
    # Switch to enable partial coded schemes: 1 means yes, and 0 means no
    PARTIAL_CODED=0
    
    # Path to folder containing the data folders: assume the base path is ~/
    DATA_FOLDER=dataset
    
    # If using real data, enter 1, otherwise 0
    IS_REAL=1
    
    # Dataset directory name
    # eg. ~/dataset/amazon-dataset/...
    DATASET=amazon-dataset        # THIS PARAM ALSO SPECIFIES THE BEHAVIOR OF PREPROCESSING THE DATA, ie. check make arrange_real_data
    N_ROWS=26210		# num of input samples, ie. X1, X2, X3... Xd
    N_COLS=241915		# num of features per input, ie. x1, x2, x3... xp



Step 3: Preprocess the dataset
------------------------------

Now, given the correct data structure on the master node as shown in the above, we will first preprocess and split data into train and test group.
Inside the gradient_coding_mds/ directory, do:

.. code-block:: 

    make arrange_real_data

If everything runs correctly, now one should expect the directory tree looks like this::

    ~/
    ├── cluster_dir_setup.sh    
    ├── copy.sh                 
    ├── dataset                 
    │   └── amazon-dataset
    |       ├── test.csv        
    |       ├── train.csv
    |       └── 10              # NEW: this directory contains the data partitions: 1~10 that will be shared to each worker
    │           ├── 1.npz
    │           ├── 10.npz
    │           ├── 2.npz
    │           ├── 3.npz
    │           ├── 4.npz
    │           ├── 5.npz
    │           ├── 6.npz
    │           ├── 7.npz
    │           ├── 8.npz
    │           ├── 9.npz
    |           ├── label.dat        # the y labels of train set
    │           ├── label_test.dat   # the y labels of test set
    |           └── test_data.npz    # the features of test set
    ├── gradient_coding_mds/    
    │   ├── main.py
    │   └── ...
    └── hello_world.py          

Step 4: Setup the working directories for each worker
-----------------------------------------------------
On the master node, then we call the following script to set up a similar working directory for each worker, to enable the MPI protocol function correctly: 

.. code-block:: 

    source cluster_dir_setup.sh

Step 5: Run the simulation
--------------------------
Finally, if everything went well, one should be able to run the simulation without a problem:

.. code-block:: 

    make naive    # you are encouraged to try other command as specified in the Makefile

And a sample output on the terminal::

    mpirun -np 11 -H localhost,w1,w2,w3,w4,w5,w6,w7,w8,w9,w10 python3 main.py 11 26210               241915          dataset 1 amazon-dataset 0 1 0 0
    ---- Starting Naive Iterations ----
             >>> At Iteration 0
             >>> At Iteration 10
             >>> At Iteration 20
             >>> At Iteration 30
             >>> At Iteration 40
             >>> At Iteration 50
             >>> At Iteration 60
             >>> At Iteration 70
             >>> At Iteration 80
             >>> At Iteration 90
    Total Time Elapsed: 15.752
    Iteration 0: Train Loss = 0.548, Test Loss = 0.555, AUC = 0.520, Total time taken =0.103
    Iteration 1: Train Loss = 0.474, Test Loss = 0.480, AUC = 0.534, Total time taken =0.240
    Iteration 2: Train Loss = 0.385, Test Loss = 0.392, AUC = 0.549, Total time taken =0.287
