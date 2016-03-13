# Zimbra 8.6.0 on CentOS 7

## Introduction
The following guide will walk you through the installation of the Zimbra mail server on CentOS 7 minimal.

The guide assumes the following information from the test system.  Substitute in your information.
```
Domain		: example.com
Hostname	: mail
IP Address	: 192.168.10.15
```
## Configure the Network Adapter

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
## Disable SELinux & iptables
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

## Configure /etc/hosts, /etc/resolv.conf and hostname

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

## Disable sendmail and postfix
Run the following commands to ensure any existing mail services are disabled.
```
sudo service sendmail stop
sudo service postfix stop
sudo systemctl disable sendmail
sudo systemctl disable postfix
```

## Update repositories and install dependencies
```
sudo yum update
sudo -y install perl perl-core wget screen w3m elinks openssh-clients openssh-server bind bind-utils unzip nmap sed nc sysstat libaio rsync telnet aspell
```
## Configure the Local DNS Server

Zimbra needs to be able to lookup the MX records on the domain it's using.  For that purpose, we can configure the DNS on Zimbra mail server.

Open file `/etc/named.conf` and add any on `listen-on port 53` and `allow-query` as follows:
```
listen-on port 53 { 127.0.0.1; any; };
allow-query     { localhost; any; };
```
Create a zone on the bottom of `named.conf` as follows:
```
zone "example.com" IN {
type master;
file "db.example.com";
allow-update { none; };
};
```
Create the database for the new zone in `/var/named`
```
touch /var/named/db.example.com
chgrp named /var/named/db.example.com
nano /var/named/db.example.com
```
Insert the following into the file.
```
$TTL 1D
@       IN SOA  ns1.example.com. root.example.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@       IN      NS      ns1.example.com.
@       IN      MX      0 mail.example.com.
ns1     IN      A       192.168.10.15
mail    IN      A       192.168.10.15
```
## Restart the DNS service and make sure it's working right
```
service named restart
systemctl enable named
nslookup mail.example.com
dig example.com mx
```
The results from the dig command above should look like this if everything is working right
```
[root@mail opt]# nslookup mail.example.com
Server:         192.168.10.15
Address:        192.168.10.15#53
Name: mail.example.com
Address: 192.168.10.15
```
All the configuration for Zimbra is complete.  Now we can install Zimbra.

## Zimbra Install

First we need to download the Zimbra Binaries from the official [link](https://files.zimbra.com/downloads/8.6.0_GA/zcs-8.6.0_GA_1153.RHEL7_64.20141215151110.tgz).
```
cd /opt/
wget -c https://files.zimbra.com/downloads/8.6.0_GA/zcs-8.6.0_GA_1153.RHEL7_64.20141215151110.tgz
```
After the download is complete, extract and install as follows:
```
tar -xzvf zcs-8.6.0_GA_1153.RHEL7_64.20141215151110.tgz
cd zcs-8.6.0_GA_1153.RHEL7_64.20141215151110.tgz
sudo sh install.sh
```
First off you have to agree to the [license agreement](https://files.zimbra.com/website/docs/8.6/open_source_licenses_8.6.0.txt).
```
Do you agree with the terms of the software license agreement? [N] Y
```
Select the packages to install as follws
```
Install zimbra-ldap [Y] Y
Install zimbra-logger [Y] Y
Install zimbra-mta [Y] Y
Install zimbra-dnscache [Y] N
Install zimbra-snmp [Y] Y
Install zimbra-store [Y] Y
Install zimbra-apache [Y] Y
Install zimbra-spell [Y] Y
Install zimbra-memcached [Y] Y
Install zimbra-proxy [Y] Y
```
Type Y if asked `The system will be modified.  Continue?`

If you recieve a message similar to the one below, type `Yes` and change the domain name.
```
DNS ERROR resolving MX for mail.example.com
It is suggested that the domain name have an MX record configured in DNS
Change domain name? [Yes] Yes
Create domain: [mail.example.com] example.com
```
If you do not change the domain name in the above section, your domain name will become mail.example.com with an email account, user@mail.mail.example.com. Type `6` and then press `ENTER` to change the password of the admin account, then type `4` and press `ENTER`. Enter your password.
````
 1) Common Configuration:                                                  
   2) zimbra-ldap:                             Enabled                       
   3) zimbra-logger:                           Enabled                       
   4) zimbra-mta:                              Enabled                       
   5) zimbra-snmp:                             Enabled                       
   6) zimbra-store:                            Enabled                       
        +Create Admin User:                    yes                           
        +Admin user to create:                 admin@example.com           
******* +Admin Password                        UNSET                         
        +Anti-virus quarantine user:           virus-quarantine.dgnsq8ewc@example.com
......
......
Address unconfigured (**) items  (? - help) 6
Store configuration

   1) Status:                                  Enabled                       
   2) Create Admin User:                       yes                           
   3) Admin user to create:                    admin@example.com           
** 4) Admin Password                           UNSET                         
   5) Anti-virus quarantine user:              virus-quarantine.dgnsq8ewc@example.com
......
......
Select, or 'r' for previous menu [r] 4

Password for admin@example.com (min 6 characters): [s8eNUeOms] Verys3cr3t
```
After entering your password, select `r` for the previous menu.  If everything has been configured correctly, apply the configuration and wait until Zimbra finishes installing.
```
*** CONFIGURATION COMPLETE - press 'a' to apply
Select from menu, or press 'a' to apply config (? - help) a
Save configuration data to a file? [Yes] Yes
Save config in file: [/opt/zimbra/config.24648] 
Saving config in /opt/zimbra/config.24648...done.
The system will be modified - continue? [No] Yes
Operations logged to /tmp/zmsetup10052014-214606.log
```
Type `Yes` if asked to `Notify Zimbra of your installation?` Your Zimbra Installation has now completed. 

Check the status of all the Zimbra services
```
su - zimbra
zmcontrol status
```
You can now access the webmail admin console with your favorite browser at the following address
```
https://mail.example.com:7017
```