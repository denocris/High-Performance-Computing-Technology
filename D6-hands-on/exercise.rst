INFO
=====


To login on the COSINT infrastucture
::

  ssh mhpcXX@hpc.c3e.cosint.it

where the XX stands for the number of your group, specified in the following table.

The accounts are assigned as follows:

+---------+----------+--------------------------------+
|  GROUP  |   USER   |  MEMBERS                       |
+=========+==========+================================+
| group1  |   mhpc01 | mowais, sparonuz, ansuini      |
+---------+----------+--------------------------------+
| group2  |   mhpc02 | raversa, ndemo, makweba        | 
+---------+----------+--------------------------------+
| group3  |   mhpc03 | igirardi, mbrenesn, plabus     |
+---------+----------+--------------------------------+
| group4  |   mhpc04 | pdicerbo, jcarmona, aando      |
+---------+----------+--------------------------------+


D2 exercise
===========

For this exercise we will evaluate the overhead of virtualization, using the xhpl benchmark, analyzing 2 different situations:

  - Run xhpl on physical machine using 8 cores and 16GB of ram compared to a virtual machine with 8 cores and 16GB ram
  - Run xhpl+gpu on physical machine using 8 cores and 16GB and of ram and one GPU compared to a virtual machine with 8 cores and 16GB ram and passthrough to 1 GPU

Group 1 and 3 will evaluate the performance of the VIRTUAL system.

Group 2 and 4 will evaluate the performance of the PHYSICAL systems.

Copy the 'hpl-lab' from /opt/cluster/mhpc

To work on the PHYSICAL machine, simply submit a job to
::

  qsub -q gpu -l walltime=12:00:00 -l nodes=bXX:ppn=24 -I 

where XX is assigned as

+---------+------------------+
|  XX     |   USER           |
+=========+==================+
| 21      |   mhpc02         | 
+---------+------------------+
| 22      |   mhpc04         |
+---------+------------------+


To login on the VIRTUAL machine
::

  ssh <group-username>@hpc.c3e.cosint.it
  ssh centos@10.2.33.XX

+---------+------------------+
|  XX     |   USER           |
+=========+==================+
| 14      |   mhpc01         | 
+---------+------------------+
| 13      |   mhpc03         |
+---------+------------------+

