Lab 6: Installing SLURM as LRMS  
---------------------------------------------------------------------


Goal
-----

Install and configure SLURM package and compare with Torque/Maui 


Tutor 
------

S.Cozzini/ G.Gallizia

Steps:
-------

  Installing Munge
  Building Slurm 
  Installing Slurm     
  Configuring Slurm 
  Testing Slurm
  Write a report and improve documentation

Resources 
----------

  http://slurm.schedmd.com/ 


Installation guide
-------------------

This below a rough and dirty guide to use as reference. It should be completed at the end of the exercise.


Installing MUNGE
^^^^^^^^^^^^^^^^

Official install guide for MUNGE:

    https://code.google.com/p/munge/wiki/InstallationGuide

The first thing to install is MUNGE. The installation needs the
``rpm-build`` package::

    yum install rpm-build

MUNGE has the following build-time dependencies::

    yum install bzip2-devel gcc openssl-devel zlib-devel 

After issuing the ``rpmbuild`` command from the guide go to
``./rpmbuild/RPMS/x86_64`` directory to find the RPMs (replace the last
directory with the right architecture).

Install the RPMs::

    rpm -ivh munge-libs-0.5.11-1.el6.x86_64.rpm munge-0.5.11-1.el6.x86_64.rpm

The command line before assumes that you're installing MUNGE 0.5.11-1,
replace the version accordingly.

Remember to install the RPMs on all the nodes, copy the file
``/etc/munge/munge.key`` (preserving ownership and permissions) and then
you can enable the ``munge`` service::

    chkconfig munge on
    service munge start

Building SLURM
^^^^^^^^^^^^^^
Super quick start:

    https://computing.llnl.gov/linux/slurm/quickstart_admin.html

To build slurm RPMs with ``rpmbuild`` you have to install the
``munge-devel`` RPM and also you have to satisfy some other
dependencies::

    rpm -ivh munge-devel-0.5.11-1.el6.x86_64.rpm
    yum install pam-devel perl-ExtUtils-MakeMaker readline-devel

Then build the RPMs with::

    rpmbuild -ta slurm-14.11.2.tar.bz2

Installing SLURM
^^^^^^^^^^^^^^^^

If you have successfully built the SLURM RPMs::

    rpm -ivh slurm-14.11.2-1.el6.x86_64.rpm \
        slurm-devel-14.11.2-1.el6.x86_64.rpm \
        slurm-pam_slurm-14.11.2-1.el6.x86_64.rpm \
        slurm-munge-14.11.2-1.el6.x86_64.rpm \
        slurm-plugins-14.11.2-1.el6.x86_64.rpm \
        slurm-perlapi-14.11.2-1.el6.x86_64.rpm \
        slurm-sjobexit-14.11.2-1.el6.x86_64.rpm

Configuring SLURM
^^^^^^^^^^^^^^^^^

To configure SLURM there's an HTML form located in
``/usr/share/doc/slurm-14.11.2/html/configurator.html`` with all the
options or you can use the
``/usr/share/doc/slurm-14.11.2/html/configurator.easy.html`` that
assumes the default value for most of the configuration options.

Both HTML files are fully annotated and can be opened with any
JavaScript capable browser. When you've finished with the configuration
press the Submit button at the bottom copy the output to a file and you
have a slurm.conf file that you tweak further or just install on the
nodes.

Before starting SLURM there are some operations that must be done:

#. You have to create a ``slurm`` user on **ALL** the nodes.

#. You have to create all the directories needed by SLURM and you have
to transfer the ownership of those directories to the above mentioned
``slurm`` user. Check your ``slurm.conf`` for the directories to create.

#. Copy the generated configuration file to ``/etc/slurm/slurm.conf``



