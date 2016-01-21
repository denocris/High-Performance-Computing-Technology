===========
D1-Hands-on
===========

To access the OpenStack dashboard
::

  http://172.16.254.104/dashboard

The credential to access the dashboard are assigned as follows:

+---------+----------+----------+-----------+
|  GROUP  |   USER   | PASSWORD |  MEMBERS  |
+=========+==========+==========+===========+
| group1  |   mhpc01 | mhpc01   | mowais    |
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

To access the OpenStack dashboard and the virtual machine you need to connected either with ethernet connection or to connect to the hidden wireless network 
::

  MHPC

with password
::

  2014-2015

==========================
Elasticluster installation
==========================

To install elasticluster you have to issue
::

  pip install ansible==1.3.3
  pip install elasticluster

You may want to install it in a virtual environment
::
  
  (pip install virtualenv)
  virtualenv elastic
  . ~/elastic/bin/activate
  
To exit the virtual environment
::

  deactivate  

