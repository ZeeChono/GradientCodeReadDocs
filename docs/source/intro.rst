Introduction
============

Gradient coding is a distributed computing technique aiming to provide robustness against slow or non-responsive 
computing nodes, known as stragglers, while balancing the computational load for responsive computing nodes. 
Among existing gradient codes, many iterations of algorithm upgrades have been seen and claimed that they 
are the best for targeted application scenarios. However, only a little of the work has tried to bring these 
valuable techniques to real-world applications such as cloud-distributed computation. In this project, we will 
set up a real-life cloud machine learning training application based on AWS services and apply the gradient 
coding idea to implementations. By inspiration from the work of Rashish Tandon and Qi Lei [Gradient-Coding]_, 
we will first reproduce their existing results of exact recovery gradient code (FRC, CRC) to understand the 
logics. Moreover, this project further extends the evaluation metrics of the gradient coding from approximation 
error (theoretically) to more tangible metrics (model accuracy etc.), making the testing process more persuasive.
Finally, we will finish a fully customized testing suite to help researchers to test any gradient coding algorithms.

**keywords:** Gradient Coding, Distributed computation, MPI protocol, Python4mpi;

**encoding schemes:** FRC(Full repetition code), CRC(Cyclic repetition code), BIBD(Balanced incomplete block design), SPG(Sparse Gradient Code)


What is MPI?
------------

MPI, [mpi-using]_ [mpi-ref]_ the *Message Passing Interface*, is a
standardized and portable message-passing system designed to function
on a wide variety of parallel computers. The standard defines the
syntax and semantics of library routines and allows users to write
portable programs in the main scientific programming languages
(Fortran, C, or C++).

Since its release, the MPI specification [mpi-std1]_ [mpi-std2]_ has
become the leading standard for message-passing libraries for parallel
computers.  Implementations are available from vendors of
high-performance computers and from well known open source projects
like MPICH [mpi-mpich]_ and `Open MPI` [mpi-openmpi]_.


What is Python?
---------------

Python is a modern, easy to learn, powerful programming language. It
has efficient high-level data structures and a simple but effective
approach to object-oriented programming with dynamic typing and
dynamic binding. It supports modules and packages, which encourages
program modularity and code reuse. Python's elegant syntax, together
with its interpreted nature, make it an ideal language for scripting
and rapid application development in many areas on most platforms.

The Python interpreter and the extensive standard library are
available in source or binary form without charge for all major
platforms, and can be freely distributed. It is easily extended with
new functions and data types implemented in C or C++. Python is also
suitable as an extension language for customizable applications.

Python is an ideal candidate for writing the higher-level parts of
large-scale scientific applications [Hinsen97]_ and driving
simulations in parallel architectures [Beazley97]_ like clusters of
PC's or SMP's. Python codes are quickly developed, easily maintained,
and can achieve a high degree of integration with other libraries
written in compiled languages.


.. References
.. ..........

.. [Gradient-Coding] Tandon, R., Lei, Q., Dimakis, A. G., & Karampatziakis, N. (2016). Gradient coding. arXiv preprint arXiv:1612.03301.

.. [mpi-std1] MPI Forum. MPI: A Message Passing Interface Standard.
   International Journal of Supercomputer Applications, volume 8,
   number 3-4, pages 159-416, 1994.

.. [mpi-std2] MPI Forum. MPI: A Message Passing Interface Standard.
   High Performance Computing Applications, volume 12, number 1-2,
   pages 1-299, 1998.

.. [mpi-using] William Gropp, Ewing Lusk, and Anthony Skjellum.  Using
   MPI: portable parallel programming with the message-passing
   interface.  MIT Press, 1994.

.. [mpi-ref] Mark Snir, Steve Otto, Steven Huss-Lederman, David
   Walker, and Jack Dongarra.  MPI - The Complete Reference, volume 1,
   The MPI Core.  MIT Press, 2nd. edition, 1998.

.. [mpi-mpich] W. Gropp, E. Lusk, N. Doss, and A. Skjellum.  A
   high-performance, portable implementation of the MPI message
   passing interface standard.  Parallel Computing, 22(6):789-828,
   September 1996.

.. [mpi-openmpi] Edgar Gabriel, Graham E. Fagg, George Bosilca, Thara
   Angskun, Jack J. Dongarra, Jeffrey M. Squyres, Vishal Sahay,
   Prabhanjan Kambadur, Brian Barrett, Andrew Lumsdaine, Ralph
   H. Castain, David J. Daniel, Richard L. Graham, and Timothy
   S. Woodall. Open MPI: Goals, Concept, and Design of a Next
   Generation MPI Implementation. In Proceedings, 11th European
   PVM/MPI Users' Group Meeting, Budapest, Hungary, September 2004.

.. [Hinsen97] Konrad Hinsen.  The Molecular Modelling Toolkit: a case
   study of a large scientific application in Python.  In Proceedings
   of the 6th International Python Conference, pages 29-35, San Jose,
   Ca., October 1997.

.. [Beazley97] David M. Beazley and Peter S. Lomdahl.  Feeding a
   large-scale physics application to Python.  In Proceedings of the
   6th International Python Conference, pages 21-29, San Jose, Ca.,
   October 1997.
