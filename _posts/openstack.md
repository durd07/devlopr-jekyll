Refer to http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_basic_environment.html

/Library/Preferences/VMware Fusion 
 edit networking to change the subnet of vmnet8

to use the static ip address at ubuntu  
 edit /etc/network/interfaces 
 auto eth0 
 iface eth0 inet static 
 address 10.0.0.21 
 netmask 255.255.255.0 
 gateway 10.0.0.2 
 dns-nameservers 10.0.0.2

/etc/hosts

# controller

10.0.0.11       controller

# network

10.0.0.21       network

# compute1

10.0.0.31       compute1 
 controller 
 auto eth0 
 iface eth0 inet static 
 address 10.0.0.11 
 netmask 255.255.255.0 
 gateway 10.0.0.2 
 dns-nameservers 10.0.0.2 
 network 
 auto eth0 
 iface eth0 inet static 
 address 10.0.0.21 
 netmask 255.255.255.0 
 gateway 10.0.0.2 
 dns-nameservers 10.0.0.2

auto eth1 
 iface eth0 inet static 
 address 10.0.1.21 
 netmask 255.255.255.0

Edit the /etc/network/interfaces file to contain the following:

# The external network interface

auto INTERFACE_NAMEiface INTERFACE_NAME inet manual 
        up ip link set dev IFACEupdowniplinksetdevIFACE up          down ip link set dev IFACE down 
 compute1 
 auto eth0 
 iface eth0 inet static 
 address 10.0.0.31 
 netmask 255.255.255.0 
 gateway 10.0.0.2 
 dns-nameservers 10.0.0.2

auto eth1 
 iface eth1 inet static 
 address 10.0.1.31 
 netmask 255.255.255.0

when run  
 openstack service create   --name keystone --description "OpenStack Identity" identity 
 error occured :  
 unsupported locale setting 
 solution: 
 export LANGUAGE=en_US.UTF-8 
 export.UTF-8 
 export LC_ALL=en_US.UTF-8 
 locale-gen en_US.UTF-8 
 dpkg-reconfigure locales

when configure neutron S 
 neutron ext-list 
 Unable to establish connection to http://controller:9696/v2.0/extensions.json

This is your host IP address: 192.168.119.101 
 This is your host IPv6 address: ::1 
 Horizon is now available at http://192.168.119.101/dashboard 
 Keystone is serving at http://192.168.119.101/identity/ 
 The default users are: admin and demo 
 The password: durd

WARNING: 
 Using lib/neutron-legacy is deprecated, and it will be removed in the future

Services are running under systemd unit files. 
 For more information see: 
 https://docs.openstack.org/devstack/latest/systemd.html

DevStack Version: rocky 
 Change: e184e762aa392047667d193b3f332665ff2e6c35 Merge "Add a note on experimental jobs" 2018-03-06 12:39:11 +0000 
 OS Version: openSUSE project 42.3 n/a

2018-03-07 10:37:28.391 | stack.sh completed in 2006 seconds.

### Vmware Player network setting

------

 

```bash
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- stop nat
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- stop dhcp
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- set vnet vmnet8 mask 255.255.255.0
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- set vnet vmnet8 addr 10.0.2.0
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- set adapter vmnet8 addr 10.0.2.2
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- set nat vmnet8 internalipaddr 10.0.2.254
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- update dhcp vmnet8
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- update nat vmnet8
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- update adapter vmnet8
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- start dhcp
c:\Program Files (x86)\VMware\VMware Player>vnetlib.exe -- start nat
```