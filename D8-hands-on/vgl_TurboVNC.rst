- yum install glx-utils

- Edit /etc/default/grub. Add “rd.driver.blacklist=nouveau nouveau.modeset=0 to “GRUB_CMDLINE_LINUX”
- grub2-mkconfig -o /boot/grub2/grub.cfg
- add the nouveau driver in the black list::
    
    echo "blacklist nouveau" > /etc/modprobe.d/blacklist.conf
    
- Reboot
- The appropriate Nvidia driver are available in ``~centos``
- Switch from graphical to text mode: ``systemctl isolate multi-user.target``
- Run the Nvidia driver installer::

    sudo sh NVIDIA-Linux-x86_64-367.57.run  -e

- Remove the nouveau driver: yum remove xorg-x11-drv-nouveau  ( this may not be installed at all, in case skip this step)
- Backup old the old initramfs: mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
- Create new initramfs: dracut -f /boot/initramfs-$(uname -r).img
- Reboot

(if kernel source is needed check: `I Need the Kernel Source <http://wiki.centos.org/HowTos/I_need_the_Kernel_Source>`__ )


Configure Xorg to use all the graphic cards::

  nvidia-xconfig --enable-all-gpus -o autodetected.xconf.conf

now backup the old X11 configuration and copy the file just created in the directory **/etc/x11**::

  mv /etc/X11/xorg.conf /etc/X11/xorg.orig
  cp autodetected.xconf.conf /etc/X11/xorg.conf

Check if the Section "Device" related to the nvidia card(s) contains the following Options::

  BusID          "PCI:00:09:0"
  Option         "UseDisplayDevice" "none"

if not, add them to all "Device" sections related to the NVidia gpu.

Install TurboVNC and VirtualGL
==============================

Download ``libjpeg-turbo``, ``VirtualGL`` and ``TurboVNC``
::

  wget -nc \
  http://downloads.sourceforge.net/project/libjpeg-turbo/1.4.2/libjpeg-turbo-official-1.4.2.x86_64.rpm \
  http://downloads.sourceforge.net/project/turbovnc/2.0.2/turbovnc-2.0.2.x86_64.rpm \
  http://downloads.sourceforge.net/project/virtualgl/2.5/VirtualGL-2.5.x86_64.rpm

Install 
::

  rpm -U libjpeg-turbo-official-1.4.2.x86_64.rpm
  rpm -U VirtualGL-2.5.x86_64.rpm
  rpm -U turbovnc-2.0.2.x86_64.rpm

Add hostname to ``/etc/hosts`` (to prevent vncserver errors)
::

  awk -v host="$HOSTNAME" '{if ($1=="127.0.0.1") print $0" "host  ; else {print $0} } ' /etc/hosts  > tmp_hosts && mv tmp_hosts /etc/hosts

Configure VirtualGL server ( answering y/y/y it typically fine )
::

  vglserver_config

VNC uses port from 5901 on (disply :1 on 5901, :2 on 5902 and so).
.. Note:: some implementations also start a basic HTTP server on port 5800+N to provide a VNC viewer as a Java applet
CentOS cloud image seems to be blocking all input traffic but SSH.
::

  iptables -I INPUT -p tcp --match multiport --dports 5900:5950 -m comment --comment " VNC ports " -j ACCEPT

For some reasons, TurboVNC executables are not in the system PATH
::

  ln -s /opt/TurboVNC/bin/vncpasswd /usr/bin/vncpasswd
  ln -s /opt/TurboVNC/bin/vncserver /usr/bin/vncserver
  ln -s /opt/TurboVNC/bin/Xvnc /usr/bin/Xvnc

Edit the SSH Server configuration to allow X forwarding::

  vim /etc/ssh/sshd_config

And change or add the following line::

  X11Forwarding yes


Finally, add ``centos`` user to vglusers by editing 
::

  /etc/groups

Starting the server
===================

To start the VNC server 
::

  vncserver

or, to select the display 
::

  vncserver :N

where ``N`` is the display number


Starting the client
===================
::

  ./vncviewer <vm-name>:1

Then, inside the VNC environment, open a terminal and issue
::

  vglrun glxgears

