Gradient Coding
===============

In many Machine Learning applications nowadays, the size of training datasets
has grown significantly over the years to the point that it
is becoming crucial to implement learning algorithms in a
distributed fashion (federated learning) [1]_. Usually, the distributed learning is setup by creating a cluster
and the central node(or master node) will split the big dataset into multiple partitions. Then after central node broadcasting the data, 
each device trains the model on its assigned data partition and only shares the model updates (like gradients) with a central server. 
The server then aggregates these updates to improve the global model. One can see that this learning system would have advantageous 
training speed up over the traditional training under big dataset, since the pressure from training is splitted over
the cluster.


Motivation
----------
In practice the gains due to parallelization are often limited due to stragglers – workers
that are slowed down due to unpredictable factors such as network latency, cpu utilization or computational 
complexity etc. [2]_ [3]_ That's where it is contributing that Rashish and Qi brought up this creative idea to apply
a theoretic framework for mitigating stragglers in distributed learning. 

In the study [4]_, the authors have shown that the AWS EC2 machines could be as 10 times slower than normal case.
So it is necessary to implement some data recovery mechanisms during the distributed learning process.The straggler 
problem is even more daunting in massive-scale computing systems such as [5]_, which use AWS Lambda. Left untreated, 
stragglers severely impact latency, as the performance in each iteration is determined by the slowest machine.

.. image:: intro/straggler_statistics.png
      :alt: Average measured communication times for a vector of dimension p = 500000 using n = 50 t2.micro worker machines (and a c3.8xlarge master machine). Error bars indicate one standard deviation.
      :width: 400px
      :height: 400px
      :align: center


Mechanisms [4]_
---------------
In Rashish's work, they focuses mainly on how to recover the returned partial gradients from each worker device. The problem
is generalized into two matrices: encoding matrix B and decoding matrix A. Imagine we have n workers and k data partitions:

.. math::

   AB = 1_{f \times k}

where f denotes the number of combinations of surviving workers/non-stragglers, :math:`1_{f \times k}` is the all 1s matrix of 
dimension :math:`f \times k`, and we have matrices :math:`A \in R^{f \times n}, B \in R^{n \times k}`.
We interpret the i-th row of B, :math:`bi`, to be the data assignment of worker i. The support of :math:`bi` corresponds
to the data partitions that the worker Wi has access to, and the entries of :math:`bi` encode a linear combination over 
their gradients that worker Wi transmits. Let :math:`g \in R^{k \times d}`, where d is the dimension of the gradient, be
a matrix with each row being the partial gradient of a data partition i.e.

.. math::

   \bar{g} = [g1,g2,...gk]^T

Then it's easy to see that the worker Wi transmits :math:`bi \bar{g}`. Notice that each worker computes a linear combination
of partital gradient and return it back to the master node. 

Since we have all possible decoding vectors stored as the rows in matrix A (Though it is more common to comput the
decoding vector in real-time). We can easily recover the full-gradient by doing dot product of decoding vector to the returned
gradients:

.. math::

   a_{i} B \bar{g} = [1,1,...,1] \bar{g} = (\sum_{i=1}^{k} g_{j})^T

Note for each iteration of training, only one decoding vector is needed as it is usually determined by the surviving
situation of the workers.

An example of encoding and decoding matrix for visually understanding:

.. math::
    A = \begin{pmatrix}
        0 & 1 & 2 \\
        1 & 0 & 1 \\
        2 & -1 & 0
        \end{pmatrix},
    and B = \begin{pmatrix}
            1/2 & 1 & 0 \\
            0 & 1 & -1 \\
            1/2 & 0 & 1
            \end{pmatrix},

It is easy for audience to check that :math:`AB=1_{3 \times 3}`. Since every row of A has exactly one zero, we say this
gradient coding scheme is robust to any one straggler.


Related Works
-------------

Before gradient coding
~~~~~~~~~~~~~~~~~~~~~~
The closest work to the idea of gradient coding is of [6]_ where Lee et al. used coding theory and treating stragglers 
as erasures in the transmission of the computed results. In contrast to Rashish's focus on codes for recovery of partial
gradients of any loss function, Lee addressed the straggler problem through techniques like: data shuffling and matrix
multiplication to solve erasures caused by straggling. Further closely, Li et al. [7]_ have shown how coding can be used 
for distributed MapReduce and how to trade off communication and computation.


Exact gradient coding
~~~~~~~~~~~~~~~~~~~~~
In the work where Rashnish and Qi brought up the idea of gradient coding, they also defined and tested two exact gradient
coding schemes namely: Fractional Repetition Scheme(FRC), Cyclic Repetition Scheme(CRC). We classify these two as the 
same category because their goals are fully recovering the exact total gradient at each iteration of learning.
The encoding matrix of FRC is defined as the following: (s stands for the number of stragglers)

.. math::
    B_{FRC} = \begin{bmatrix}
              B_{block}^{(1)} \\
              B_{block}^{(2)} \\
              ...\\
              B_{block}^{(s+1)}
              \end{bmatrix}_{n \times n}, where

.. math::
    B_{block}(n,s) = \begin{bmatrix}
                     1_{1 \times (s+1)}& 0_{1 \times (s+1)}& ...& 0_{1 \times (s+1)} \\
                     0_{1 \times (s+1)}& 1_{1 \times (s+1)}& ...& 0_{1 \times (s+1)} \\
                     ... & ... & ... & ...\\
                     0_{1 \times (s+1)}& 0_{1 \times (s+1)}& ...& 1_{1 \times (s+1)}
                     \end{bmatrix}_{n/(s+1) \times n},

The :math:`B_{block}` is a diagonal sequence of 1 matrix that repeatedly constructed the encoding matrix :math:`B_{FRC}`.
The idea is to repeat every data partition s+1 times given at most s stragglers so it is possible to always fully recover
the total gradient by doing linear combination.


CRC on the other hand, adopts the same idea. However, it does not need n to be divisible by s+1:

.. math::
    B_{CRC} = \begin{bmatrix}
              * & * & ... & * & 0 & 0 & ... & 0 \\
              0 & * & * & ... & * & 0 & ... & 0 \\
              ... & ... & ... & ... & ... & ... & ... & ... \\
              0 & 0 & ... & 0 & * & * & ... & * \\
              ... & ... & ... & ... & ... & ... & ... & ... \\
              * & ... & * & 0 & 0 & ... & 0 & * 
              \end{bmatrix}_{n \times n},

where * indicates non-zero entries in :math:`B_{CRC}`. This gradient coding scheme is more flexible to construct at encoding
stage, at the expense of solving the decoding vector in real-time each iteration. Due to the non-binary entries in :math:`B_{CRC}`,
the time it cost to compute decoding vector might be a drawback of the system.


Approximate gradient coding
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Swanand et al. [8]_ stated in their work about two major limitations of the exact gradient coding scheme:
First, exact gradient coding require heavy computational and storage overhead at each worker. 
In particular, in [4]_ , it was established that any coding scheme designed to tolerate S stragglers must have
L ≥ K(S + 1)/N (L is how many data partitions loaded to each worker, K is total data partitions, N is number of workers).
This implies that the higher the straggler tolerance required, the larger is the computation and storage
overhead per worker. 
Second, since the schemes are designed for a particular number of stragglers S, it is necessary to have an 
estimate on S at the design time. This is not feasible for many practical schemes as straggler behavior can vary
unpredictably.

Approximate grdient coding, as the name suggest, are the coding schemes that would recover the total gradient approximately. Indeed, 
in many practical learning algorithms, it is sufficient to approximately reconstruct the gradient sum. By the idea of
stochastic gradient descent, adding some noise to the gradient might help the model update in a better direction.
These approximate gradient codes do not require to have an estimate of the number of stragglers S a priori, 
and allow the computation and storage overhead per worker to be substantially small.

By proposing the gradient coding using balanced incomplete block designs(BIBD) incidence matrix as the encoding matrix,
Swanand showed that approximation error for these codes depends only on the number of stragglers, and not on which
specific set of workers is straggling (Due to the nature of the BIBD matrices, any straggling pattern would cause
similar effect). Therefore, the decoding vector at the master node can be computed in closed-form and save a lot of time.
Through the work, they have shown the symmetric BIBDs are excellent candidates for approximate gradient code.


Moreover, Rashish in another work collaborating with Netanel [9]_ , have approached to the approximate gradient code based on 
normalized adjacency matrices of expander graphs. And our lab member is working on a sparse gaussian gradient code to
mimic the effect of BIBD by equalizing the expectation value, aiming to solve the rarity of the BIBD matrices 
(BIBD's existence is limited).


Goal
----
The main goal of this project is to create a universal and automated testing suite for the researchers to try out
their gradient coding ideas. By the help of modern OOP programming language Python and AWS cloud services, we should
be able to do this. The model is straight forward, our python scripts will ask the user to provide gradient code 
encoding matrix and possible decoding vector. After providing the dataset and number of the worker nodes in a cluster,
our testing suite should be able to carry the tests multiple times, at the willing of the user, and then produce the 
output as the evaluation metrics over iteration or over time. The model is shown below:

   .. image:: intro/model.png
      :alt: model
      :width: 400px
      :height: 300px
      :align: center

Though the process is quite automatic, it does not exclude the cutomizability from the user, as the user have the 
freedom to configure the encoding and decoding rule (exact or non-exact) of the test at the first place.

.. References
.. ..........

.. [1] Li, L., Fan, Y., Tse, M., & Lin, K. Y. (2020). A review of applications in federated learning. 
   Computers & Industrial Engineering, 149, 106854.

.. [2] T. Hoefler, T. Schneider, and A. Lumsdaine, “Characterizing the influence of system noise on 
   large-scale applications by simulation,” in Proc.of the ACM/IEEE Int. Conf. for High Perf. Comp., Networking, Storage and Analysis, 2010, pp. 1–11.

.. [3] J. Dean and L. A. Barroso, “The tail at scale,” Commun. ACM, vol. 56, no. 2, pp. 74–80, Feb 2013.

.. [4] Tandon, R., Lei, Q., Dimakis, A. G., & Karampatziakis, N. (2016). Gradient coding. arXiv preprint 
   arXiv:1612.03301.

.. [5] E. Jonas, Q. Pu, S. Venkataraman, I. Stoica, and B. Recht, “Occupy the
   cloud: Distributed computing for the 99%,” in Proceedings of the 2017
   Symposium on Cloud Computing, ser. SoCC ’17, 2017, pp. 445–451.

.. [6] Lee, Kangwook, Lam, Maximilian, Pedarsani, Ramtin,
   Papailiopoulos, Dimitris S., and Ramchandran, Kannan. Speeding up distributed machine learning using
   codes. CoRR, abs/1512.02673, 2015. URL http:
   //arxiv.org/abs/1512.02673.

.. [7] Li, S., Maddah-Ali, M. A., and Avestimehr, A. S. Coded
   mapreduce. In 2015 53rd Annual Allerton Conference
   on Communication, Control, and Computing (Allerton),
   pp. 964–971, Sept 2015. doi: 10.1109/ALLERTON.2015.
   7447112.

.. [8] Kadhe, S., Koyluoglu, O. O., & Ramchandran, K. (2019, July). Gradient coding based on block designs for 
   mitigating adversarial stragglers. In 2019 IEEE International Symposium on Information Theory (ISIT) (pp. 2813-2817). IEEE.

.. [9] Raviv, N., Tandon, R., Dimakis, A., & Tamo, I. (2018, July). Gradient coding from cyclic MDS codes and 
   expander graphs. In International Conference on Machine Learning (pp. 4305-4313). PMLR.

