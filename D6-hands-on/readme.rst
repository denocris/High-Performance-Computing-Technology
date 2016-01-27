Installing a Lustre FileSystem
===================================================================

Goal
-----

Goal of the exercise is to install the parallel and distributed Lustre filesystem, using some additional HW to act as I/O servers.

For any further information please see Lustre Operational Manual at::

	https://build.hpdd.intel.com/job/lustre-manual/lastSuccessfulBuild/artifact/lustre_manual.pdf


Steps
------------
 
  - install Lustre components
  - configure storage 
  - benchmarks storage (iozone) 
  - install Lustre FS 
  - install lustre client  
  - test and benchmark the Lustre FS 
  - make a short report, cleaning and completing this document


Install the Lustre components
===============================

Lustre is build up on a server part and a client part. Intel releases the server part as a Linux patched kernel.

The server part is configured on different components:

- the MGS, ManaGement Server
- the MDS, MetaData Server
	- it stores metadata in a MDT, a MetaData Target
- the OSSs, Object Storage Server
	- they store data in the OSTs, the Object Storage Target

The client part is released as different modules for Linux kernel (namely the Lustre `patchless client`).

You need hardware for each of the above components. 

You will install 4 machines, with the following configuration

+------------+---------+++-+----------+
|            | services    | targets  |
+============+=============+==========+
| server1    | MGS,MDS,OSS | MDT, OST |
+------------+-------------+----------+
| server2    | OSS         | OST      |
+------------+-------------+----------+
| client1    | client      |          |
+------------+-------------+----------+
| client2    | client      |          |
+------------+-------------+----------+


----

First step will be install the latest maintenance release.

Actually last Lustre version is 2.5.3. As from version 2.5.x, Intel provides ready to use repositories, so a yum repository has been created on the `marvin` machine (ip 172.17.0.1).
`ciccio2` exports the `/distro` directory over the network, so you need to mount that directory on the Lustre server and then configure yum accordingly.
	
Enter the `/etc/yum.repos.d` directory and create a new entry to point to the right repository. You may want to set the gpgcheck to 0.

Once yum is correctly configured and the new repositories have been registered in yum database, proceed with the Lustre server installation
::

  yum install lustre-tests
	
Take a look on every package yum wants to install and try to identify them.
Yum will probably complain about the kernel, since you are trying to install an older one. To work around this, you need to manually force an install of the kernel RPMs directly from command line (_install_ the kernel RPM and _upgrade_ the kernel-firmware RPM you find in /distro/lustre/server/), then retry with yum.

Once yum has finished to install all the RPMs you need, reboot the server. Why a reboot is needed?

On the servers you will also need the `e2fsprog` tools. They will be installed automatically with the above yum command, but in case of needs you will find further info here
::
  
  https://wiki.hpdd.intel.com/display/PUB/Lustre+Tools


You may want to configure the LNET now. `LNET` is the ``lustre networking``, a kernel module in charge of the networking layer in Lustre.

You need to instruct Lustre on which layer you are going to use to transfer I/O over the network.

What kind of transport is available on your machine?

Edit `/etc/modprobe.d/lustre.conf` and, for example, to use `TCP` on the nic `eth1`
::

  options lnet networks=tcp0(eth1)
	
or to use InfiniBand
::

  options lnet networks=o2ib0(ib0)
	
In order to use the InfiniBand layer you need to create an IPoIB configuration. Lustre uses RDMA but communication starts over TCP/IP. 

The RDMA service must be running on the servers, and you may want to have it running on startup
::

  chkconfig add rdma
  chkconfig rdma on

Reboot the server.

You need now to configure the Infiniband interface. Edit `/etc/sysconfig/network-scripts/ifcfg-ib0` in order to have a static ip assigned to the ib0 interface. 

The ip you need to use is 
::

  10.3.X.Y

where X is the number of your cluster, from 1 to 4, and Y indicates the node of the cluster, again from 1 to 4. 


----

Try now to understand if the Lustre networking is working as expected. You may want to use the provided interface `lctl` (``lustre control``).

For instance, if you have two Lustre servers configured, you can try to list the network configured on each server
::

  lctl list_nids
	
And then to ping from one to the other::
::
  
  lctl ping [nid of another Lustre server]

Note: You may need to load the lustre kernel module using 
::

  modprobe lustre

This module will be later loaded automatically when you mount a lustre filesystem.

---- 

On the client side, after mounting /distro as for the servers, you need to run::
::
  
  yum install lustre-client
        
As for the server, you need to configure the the Infiniband interface, editing `/etc/sysconfig/network-scripts/ifcfg-ib0`, with the same logic as before. 

Moreover, as done on the servers, you need to define the network lustre.

Edit `/etc/modprobe.d/lustre.conf` adding  
::

  options lnet networks=o2ib0(ib0)

----




Configure storage on servers
===============================

For each component of the Lustre server part you need a storage (a Lustre _target_) to store data and metadata.

In this part you will identify the devices you will use for Lustre targets, benchmark and format them using Lustre `mkfs` tool.

Identify some disk space not used, or not necessary to run the operating system or other services.

Once you identified the targets, you may want to proceed with a short benchmarking activity on the raw storage.

Lustre packages provide the `sgpdd-survey` tool and the `obd-survey` tool to understand devices performance. Be aware that these tools are low level and, hence, destroy data.

Run `sgpdd-survey` on the Lustre targets and save the results you get. 

The goal is to understand how targets react when you use
	- single and multiple thread over a single target
	- multiple thread over multiple targets

The numbers you will get will be useful to know how much in performance you will lose when you will produce I/O in the final filesystem.

Finally, run `mds-survey` to simulate Lustre metadata performance. Find further information here::

	http://www.eofs.eu/fileadmin/lad2014/slides/03_Shuichi_Ihara_Lustre_Metadata_LAD14.pdf
	


----

Proceed with format operations.

Before formatting, you need to create lvm volumes inside the lvm volume group.
::

  lvcreate -L 5G -n mdt VolGroup0

will create a volume a volume called `mdt` in the volume group VolGroup0 of size 5GB
::

  lvcreate -l 100%FREE -n ost VolGroup0 

will create a volume called `ost` in the volume group VolGroup0 using all the available space in VolGroup0


To list physical volume, volume groups and logical volume you can use, respectively
::

  pvdisplay
  vgdisplay
  lvdisplay


The following is a command line to create a combined MGS/MDT service::

	mkfs.lustre --verbose --mgs --mdt --fsname=lustre --backfstype ldiskfs --index --reformat /dev/[your target]
	
While with the flag `--ost` you create an OST target::

	mkfs.lustre --verbose --ost --fsname=lustre --mgsnode=[your MGS machine]@[network to reach MGS] --backfstype ldiskfs --index --reformat /dev/[your target]


Note: Remeber that server1 will have mgs, mds and oss, but server2 will have oss only. 

----

You can try now to mount the target on the Lustre servers::

	mount -t lustre /dev/[your target] /[your mount point]
	
In case of errors look at `dmesg` command to understand what is going wrong.

Mount the filesystem on clients
==================================

Once Lustre targets have been formatted, you can try to mount the filesystem on the clients::

	mount -t lustre [name or address of MGS]@[network to reach the MGS]:/<lustre_filesystem_name> <folder in which you want to mount>

The network is the one defined in `etc/modprobe.d/lustre.conf`, so `o2ib0`, while the lustre_filesystem_name is the the name you defined in the option `--fsname` for mkfs.lustre. 

In case of errors look at `dmesg` command to understand what is going wrong.


Testing the filesystem
==========================

Intel releases the `lfs` interface to control and monitor the Lustre filesystem, from a Lustre client.

Take a look on `lfs find`, `lfs df`, etc.

Try to understand how `lfs getstripe` and `lfs setstripe` work.

Benchmark the filesystem and report the results. You can you use the following tools:

	- ost-survey, relased with Intel Lustre packages, to stress OSTs
	- iozone to create I/O on the filesystem with multiple threads, will stress OSTs
	- IOR to create I/O on the filesystem with different stripe size, will stress OSTs
	- mdtest to stress the MDT
	
Compare the results you had when running benchmark at low level (`sgpdd-survey` and `obd-survey`) and try to understand the overhead of the filesystem on the disk and the network.

If time permits, plot values from both `sgpdd-survey` and `iozone`.
