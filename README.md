# sles12sp3devstack

Getting devstack to deploy on sles12sp3 
This site contains links to patches and instructions on how to deploy devstack on a sles12 sp3 server. 


### Introduction
In order to install the correct packages onto the sles12sp3 server we need to configure the appropriate software Repositories or 'repos'
The easies way is really to create an iso of the software necessary and use the iso files as the url for the repository. To add the repositories to the zypper configuration of the sles12sp3 server, we can use the either the 'yast' command or the "zypper ar ..." command. I have found it easier to use yast just because it avoids typos and such. 

Depending on the version of devstack you might have to use the following command 
```
stack@dev:/opt/stack/devstack> FORCE=yes ./stack.sh 
```

Sles12sp3 comes with MariaDB, which is the prefered databse solution in a SLES deployment. However devstack is usually deployed onto a mysql database (the default).  In order to not confuse the subject we'll split the task into two different sections. 
- One: deploy devstack on sles12sp3 using a mysql backend
- Two: deploy devstack on sles12sp3 using a MariaDB backend


### Zero: install and configure sles12 sp3 for devstack
Basically install sles12 sp3 and configure as you would for any other deployment, but for SUSE :). 
This place is got pretty good instructions: http://nubla.do/devstack-opensuse/

A quick outline
- Install Git
- Configure stack user correctly. 
- Install some packages
- Clone devstack repo
- Stack


NOTE: in sles12sp3 the "sudo" group is the "wheel" group.

### One: devstack on sles12 sp3 using mysql backend

#### Instsall basic repositories
We'll start with three repositories to add. Ideally we should remove all other respositories just to be safe that we don't pull the wrong pacakge version from somewhere unintended. Removing the respositories is not a must, but recommended. 

We also assume that we have access to the sles12sp3 server iso and sles12sp3sdk iso. In the example below they have been "inserted" into cdrom 0 and cdrom 1. In reality the example comes from a vm, so the isos have been connected to the vm using virt-manager. 

`sudo zypper ar https://download.opensuse.org/repositories/Cloud:/OpenStack:/Newton/SLE_12_SP3/ CloudSP3`

To mount the iso as repositories you should use yast. I will not even attempt to put the commands because they simply would be wrong. Just use yast!

At the end your repositories shoudl look something like so:

```  
 # | Alias                              | Name                                 | Enabled | GPG Check | Refresh | URI
---+------------------------------------+--------------------------------------+---------+-----------+---------+--------------------------------------------------------------------------------
 1 | CloudSP3                           | CloudSP3                             | Yes     | (r ) Yes  | No      |https://download.opensuse.org/repositories/Cloud:/OpenStack:/Newton/SLE_12_SP3/
 2 | SDK12-SP3_12.3-0                   | SDK12-SP3 12.3-0                     | Yes     | (r ) Yes  | No      | cd:///?devices=/dev/sr2
 3 | SLES12-SP3_12.3-0                  | SLES12-SP3 12.3-0                    | Yes     | (r ) Yes  | No      | cd:///?devices=/dev/sr1
 ```

#### Install some packages

```
sudo zypper install -y git
sudo zypper install -y lsb-release
sudo zypper install -y python-os-testr
```

#### Install pip
```
wget https://bootstrap.pypa.io/get-pip.py
sudo python ./get-pip.py
```


#### Install mysql repositories. 
```
wget https://dev.mysql.com/get/mysql57-community-release-sles12-11.noarch.rpm
sudo rpm -Uvh mysql57-community-release-sles12-11.noarch.rpm
sudo rpm --import /etc/RPM-GPG-KEY-mysql
```
#### Clone devstack and patch
(hopefully at some point this patch or one similar merges):
https://review.openstack.org/#/c/505433/ 

```
sudo mkdir /opt/stack
sudo chown stack /opt/stack
cd /opt/stack
git clone https://git.openstack.org/openstack-dev/devstack
git fetch https://git.openstack.org/openstack-dev/devstack refs/changes/33/505433/2 && git cherry-pick FETCH_HEAD
```
#### Stack
we'll use a basic local.conf here:
```
cat /opt/stack/devstack/local.conf

[[local|localrc]]
ADMIN_PASSWORD=secret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```

Stack using FORCE=yes (disables distro requirement)
```
FORCE=yes /opt/stack/devstack/stack.sh
```

