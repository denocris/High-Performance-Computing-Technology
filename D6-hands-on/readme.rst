============================
D6-Hands-on: Openstack 
===========================

To access the CNR/IOM OpenStack dashboard
::

  http://nimbo.escience-lab.it/dashboard

The credential to access the dashboard are assigned as follows:

+---------+----------+----------+-----------+
|  GROUP  |   USER   | PASSWORD |  MEMBERS  |
+=========+==========+==========+===========+
| group1  |   mhpc01 | mhpc01   |           |
+---------+----------+----------+-----------+
|         |   mhpc02 | mhpc02   | sparonuz  |
+---------+----------+----------+-----------+ 
|         |   mhpc03 | mhpc03   | aansuini  |
+---------+----------+----------+-----------+
| group2  |   mhpc04 | mhpc04   | raversa   |
+---------+----------+----------+-----------+
|         |   mhpc05 | mhpc05   | ndemo     |
+---------+----------+----------+-----------+
|         |   mhpc06 | mhpc06   | makweba   |
+---------+----------+----------+-----------+
| group3  |   mhpc07 | mhpc07   | igirardi  |
+---------+----------+----------+-----------+
|         |   mhpc08 | mhpc08   | mbrenesn  |
+---------+----------+----------+-----------+
|         |   mhpc09 | mhpc09   | plabus    |
+---------+----------+----------+-----------+
| group4  |   mhpc10 | mhpc10   | pdicerbo  |
+---------+----------+----------+-----------+
|         |   mhpc11 | mhpc11   | jcarmona  |
+---------+----------+----------+-----------+
|         |   mhpc12 | mhpc12   | aando     |
+---------+----------+----------+-----------+


To login on the virtual machines
::

	ssh centos@<ip>


Note
====

To access the OpenStack dashboard and the virtual machine you need  to be connected either with ethernet connection or to connect to the hidden wireless network 
::

  MHPC

with password
::

  2014-2015

==========================
Elasticluster installation
==========================


Do::

 sudo apt-get install gcc g++ git libc6-dev libffi-dev libssl-dev python-dev virtualenv

or::

 yum install gcc gcc-c++ git libffi-devel openssl-devel python-devel python-virtualenv

Create a virtualenv and clone and install::

 virtualenv elasticluster
 . elasticluster/bin/activate
 cd elasticluster
 git clone git://github.com/gc3-uzh-ch/elasticluster.git src
 cd src
 pip install -e .

Then::

 elasticluser start slurm

 elasticluster list

 elasticluster stop slurm


To exit the virtual environment
::

  deactivate  

