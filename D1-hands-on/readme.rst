===========
D1-Hands-on
===========

To access the OpenStack dashboard
::

  http://172.16.254.104/dashboard

The credential to access the dashboard are assigned as follows:

+---------+----------+----------+--------------------------------+
|  GROUP  |   USER   | PASSWORD |  MEMBERS                       |
+=========+==========+==========+================================+
| group1  |   mhpc01 | mhpc01   | mowais, sparonuz, ansuini      |
+---------+----------+----------+--------------------------------+
| group2  |   mhpc02 | mhpc02   | raversa, ndemo, makweba        | 
+---------+----------+----------+--------------------------------+
| group3  |   mhpc03 | mhpc03   | igirardi, mbrenesn, plabus     |
+---------+----------+----------+--------------------------------+
| group4  |   mhpc04 | mhpc04   | pdicerbo, jcarmona, aando      |
+---------+----------+----------+--------------------------------+


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

