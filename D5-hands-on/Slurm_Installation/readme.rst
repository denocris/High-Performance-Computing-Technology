Lab 6: Installing SLURM as LRMS
-------------------------------

Goal
----

Install and configure SLURM package and compare with Torque/Maui.

Tutor 
-----

S.Cozzini/ G.Gallizia

Steps:
------

  #. Prerequisites
  #. (Optional) Install pdsh
  #. Install MUNGE
  #. Build Slurm
  #. Install Slurm
  #. Configure Slurm
  #. Test Slurm
  #. Write a report and improve documentation

Resources
---------

  http://slurm.schedmd.com/

Prerequisites
-------------

  - A CentOS 6.5 instance with a floating IP associated to use as master
    node and two CentOS 6.5 instances without a floating IP associated
    to use as compute nodes.
  - Passwordless authentication for (at least) ``root`` on all the
    cluster nodes.
  - A working DNS *or* a single ``/etc/hosts`` file with all the cluster
    nodes copied on every node of the cluster.
  - (Optional) Shared ``/home`` and ``/opt`` filesystems from master to
    compute nodes.

Passwordless authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once a user authenticates on the master node as ``root`` she must be
able to log in on the nodes without having to type a password every time
she logs in. These instructions should work on the OpenStack virtual
environment of the laboratory.

#. Log into the master node then become the superuser with::

        $ ssh-add your_ssh_key
        $ ssh -A cloud-user@$MASTER
        $ sudo -i

   Don't forget to substitute ``your_ssh_key`` with the path to your
   *private* SSH key. The ``-A`` switch enables SSH agent forwarding and
   ``$MASTER`` should be changed with the effective IP address of the
   master node when you type the ``ssh`` command.

#. Generate a couple of SSH keys without a passphrase::

        # ssh-keygen -t rsa -C "Passwordless login key"
        Generating public/private rsa key pair.
        Enter file in which to save the key (/root/.ssh/id_rsa):
        Created directory '/root/.ssh'.
        Enter passphrase (empty for no passphrase):
        Enter same passphrase again:
        Your identification has been saved in /root/.ssh/id_rsa.
        Your public key has been saved in /root/.ssh/id_rsa.pub.
        The key fingerprint is:
        4c:80:00:ed:aa:93:a1:bf:38:47:1f:4e:ad:e4:3d:a4 Passwordless login key
        The key's randomart image is:
        +---[RSA 2048]----+
        |.o.. ..          |
        |  . .  .         |
        | .      .        |
        |  .    o         |
        | .   .  S        |
        |o . + o          |
        |o+ * *           |
        |*.. E o          |
        |.=o.   .         |
        +-----------------+

   When ``ssh-keygen`` asks for a password just press the enter key to
   leave it empty.

#. Append the newly created public key to ``/root/.ssh/authorized_keys``::

        # cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys

#. Pack the contents of ``/root/.ssh`` in to a tar archive::

        # cd /root
        # tar -cvf ssh.tar .ssh/

#. Copy the tar file into the cloud-user home directory and make sure it
   is world-readable::

        # cp /root/ssh.tar /home/cloud-user/
        # chmod +r /home/cloud-user/ssh.tar

#. Drop your privileges and copy ``ssh.tar`` on the compute nodes::

        # exit
        $ scp ssh.tar $NODE_IP:

   ``$NODE_IP`` must be set to the target compute node internal IP
   address.

#. Log into the compute nodes from the master node, become root and
   unpack the tar file into the ``/root`` directory::

        $ ssh $NODE_IP
        $ sudo -i
        # cp /home/cloud-user/ssh.tar /root/
        # tar -xvf ssh.tar

Question: why do we have to do all this stuff? (Hint: look at the
original ``authorized_keys`` file and try to guess what it does).

Edit and propagate the hosts file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Log into the master node as root.

#. Edit the ``/etc/hosts`` file and append the internal IP addresses for
   the master node and the compute nodes. Your file should look like
   this::

        127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
        ::1             localhost6 localhost6.localdomain6

        #Slurm nodes
        192.168.0.50   slurm-frontend
        192.168.0.51   slurm-compute1
        192.168.0.52   slurm-compute2

   Please do not blindly copy this sample into your ``/etc/hosts`` file
   or else you will deserve all the frustration a broken configuration
   will dump on you.

#. Try to log into the nodes using names instead of IPs::

        # ssh slurm-compute1
        # exit

#. If you can log into the nodes then you can propagate your
   ``/etc/hosts`` file with this oneliner::

        for i in `seq 2` ; do scp /etc/hosts slurm-compute$i:/etc/hosts ; done

(Optional) Share homes and executables via NFS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

On a real cluster the administrator has to provide a way to share some
mass storage between the master node and the compute nodes. The users
will gladly fill their homes with data and expect the compute nodes to
be able to read that data. There is also the added benefit of installing
software and libraries only inside a shared directory and make it
visible to all the compute nodes instead of installing the same software
and libraries on all the nodes of a cluster.

The easiest way to accomplish this under CentOS is by setting up NFS.

#. Log into the master node as root.

#. Install the nfs packages via yum::

        # yum install nfs-utils nfs-utils-lib

#. Create the ``/opt/cluster`` directory::

    # mkdir /opt/cluster

#. Edit ``/etc/exports`` to export ``/home`` and ``/opt/cluster``. E.G.::

        #Homes
        /home           192.168.0.0/24

        #Software and libraries
        /opt/cluster    192.168.0.0/24

#. Enable nfs service and start it::

        # chkconfig nfs on
        # service rpcbind start
        # service rpcidmapd start
        # service nfs start

#. Configure firewall::

        # iptables -I INPUT -s 192.168.0.0/24 -j ACCEPT

#. Log into the nodes as root.

#. Install the nfs packages via yum::

        # yum install nfs-utils nfs-utils-lib

#. Append lines like these to ``/etc/fstab``::

        192.168.0.50:/home             /home           nfs     defaults      0   0
        192.168.0.50:/opt/cluster      /opt/cluster    nfs     defaults      0   0

#. Create the ``/opt/cluster`` directory::

        # mkdir /opt/cluster

#. Mount the filesystems::

        # mount /home
        # mount /opt/cluster

Installation guide
------------------

This below is a rough and dirty guide to use as reference. It should be
completed at the end of the exercise.

(Optional) Install pdsh
^^^^^^^^^^^^^^^^^^^^^^^

While it is not required to have a distributed shell to run jobs on a
cluster using Slurm it is very convenient to have a distributed shell to
ease the setup and configuration.

The Parallel Distributed Shell or ``pdsh``
( https://code.google.com/p/pdsh/ ) does not require a particular
configuration file and can be installed via YUM::

    yum install epel-release
    yum install pdsh pdsh-rcmd-ssh

If the cluster nodes are named ``slurm-compute1``, ``slurm-compute2``
and ``slurm-frontend``:

- To issue a command on both ``slurm-compute01`` and ``slurm-compute2`` from
  ``slurm-frontend``::

    root@slurm-frontend:~# pdsh -R ssh -w slurm-compute[01-02] hostname

- To install ``pdsh`` on the nodes from ``slurm-frontend`` (this is needed in
  order to have ``pdcp`` available on the nodes)::

    root@slurm-frontend:~# pdsh -R ssh -w slurm-compute[01-02] yum -y install pdsh pdsh-rcmd-ssh

- To copy a file from ``slurm-frontend`` to both the nodes you can use ``pdcp``::

    root@slurm-frontend:~# pdcp -R ssh -w slurm-compute[01-02] /etc/hosts /etc/hosts

``pdsh`` manual:

    http://linux.die.net/man/1/pdsh

``pdcp`` manual:

    http://linux.die.net/man/1/pdcp

Install MUNGE
^^^^^^^^^^^^^

Official install guide for MUNGE:

    https://github.com/dun/munge/wiki/Installation-Guide

The first thing to install is MUNGE. The installation needs the
``rpm-build`` package::

    yum install rpm-build

MUNGE has the following build-time dependencies::

    yum install bzip2-devel gcc openssl-devel zlib-devel


Download ``bzip2`` archive::

    wget https://github.com/dun/munge/releases/download/munge-0.5.11/munge-0.5.11.tar.bz2

Issue::

    rpmbuild -tb --clean munge-0.5.11.tar.bz2

Go to
``./rpmbuild/RPMS/x86_64`` directory to find the RPMs (replace the last
directory with the right architecture).

Install the RPMs::

    rpm -ivh munge-libs-0.5.11-1.el6.x86_64.rpm munge-0.5.11-1.el6.x86_64.rpm

The command line before assumes that you're installing MUNGE 0.5.11-1,
replace the version accordingly.

Remember to install the RPMs on all the nodes. Copy the file
``/etc/munge/munge.key`` from the master to the nodes (preserving ownership and permissions) and then
you can enable the ``munge`` service::

    chkconfig munge on
    service munge start

Please note that if you have set up the NFS shares you can copy the RPMs
in the ``/opt/cluster`` directory to have them available on all the
nodes::

    mkdir -p /opt/cluster/rpm
    cp ./rpmbuild/RPMS/x86_64/*rpm /opt/cluster/rpm/

Building SLURM
^^^^^^^^^^^^^^
Super quick start:

    https://computing.llnl.gov/linux/slurm/quickstart_admin.html

To build slurm RPMs with ``rpmbuild`` you have to install the
``munge-devel`` RPM and also you have to satisfy some other
dependencies::

    rpm -ivh munge-devel-0.5.11-1.el6.x86_64.rpm
    yum install pam-devel perl-ExtUtils-MakeMaker readline-devel

Get the slurm tar-ball::

    wget http://www.schedmd.com/download/archive/slurm-14.11.2.tar.bz2

Then build the RPMs with::

    rpmbuild -ta slurm-14.11.2.tar.bz2

Installing SLURM
^^^^^^^^^^^^^^^^

If you have successfully built the SLURM RPMs you can install them::

    rpm -ivh slurm-14.11.2-1.el6.x86_64.rpm \
        slurm-devel-14.11.2-1.el6.x86_64.rpm \
        slurm-pam_slurm-14.11.2-1.el6.x86_64.rpm \
        slurm-munge-14.11.2-1.el6.x86_64.rpm \
        slurm-plugins-14.11.2-1.el6.x86_64.rpm \
        slurm-perlapi-14.11.2-1.el6.x86_64.rpm \
        slurm-sjobexit-14.11.2-1.el6.x86_64.rpm

Then transfer those RPMs to the compute nodes and install them.

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

**WARNING**: you have to add ALL the nodes to the ``slurm.conf``. The
configurator web page DOES NOT add the master node to the nodes list and
so you have to add it yourself.

Before starting SLURM there are some operations that must be done:

#. You have to create a ``slurm`` user on **ALL** the nodes.

#. You have to create all the directories needed by SLURM and you have
   to transfer the ownership of those directories to the above mentioned
   ``slurm`` user. Check your ``slurm.conf`` for the directories to create.

#. Copy the generated configuration file to ``/etc/slurm/slurm.conf``



