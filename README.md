# Zimbra 8.6.0 on CentOS 7

# Introduction
The following guide will walk you through the installation of the Zimbra mail server on CentOS 7 minimal.

The guide assumes the following information from the test system.  Substitute in your information.
```
Domain		: example.com
Hostname	: mail
IP Address	: 192.168.10.15

# Configure the Network Adapter

First we will configure the network adapter for CentOS.  The guide assumes you are using the `eth0` network interface.
```
sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
```
```
DEVICE=eth0
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=none
IPADDR=192.168.10.15
NETMASK=255.255.255.0
GATEWAY=192.168.10.1
DNS1=192.168.10.11
DNS2=192.168.10.12
USERCTL=no
```
Restart the network service and make sure it's turned on at boot.
```
service network restart
chkconfig network on
```
# Disable SELinux & iptables
open file `/etc/sysconfig/selinux` and change `SELINUX=enforcing` to `SELINUX=disabled`.  Also disable possible firewalls.
```
setenforce 0
service firewalld stop
service iptables stop
service ip6tables stop
systemctl disable firewalld
systemctl disable iptables
systemctl disable ip6tables
```

# Configure /etc/hosts, /etc/resolv.conf and hostname

Open `/etc/hosts` with your favorite editor and configure as follows:
```
sudo nano /etc/hosts
```
```
127.0.0.1     localhost
192.168.10.15 mail.example.com mail
```
Open `/etc/resolv.conf` with your favorite editor and configure as follows:
```
sudo nano /etc/resolv.conf
```
```
search example.com
nameserver 192.168.10.11
nameserver 192.168.10.12
nameserver 8.8.8.8
```
The following commands need to be done as root
```
sudo su -
```
```
hostname mail.imanudin.net
echo "HOSTNAME=mail.imanudin.net" &gt;&gt; /etc/sysconfig/network
```
Don't forget to `exit` out of root.

# Disable sendmail and postfix
Run the following commands to ensure any existing mail services are disabled.
```
sudo service sendmail stop
sudo service postfix stop
sudo systemctl disable sendmail
sudo systemctl disable postfix
```

# Update repositories and install dependencies
```
sudo yum update
sudo -y install perl perl-core wget screen w3m elinks openssh-clients openssh-server bind bind-utils unzip nmap sed nc sysstat libaio rsync telnet aspell
```
