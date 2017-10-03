# sles12sp3davestack

Getting devstack to deploy on sles12sp3 
This site contains links to patches and instructions on how to deploy devstack on a sles12 sp3 server. 


### Introduction
In order to install the correct packages onto the sles12sp3 server we need to configure the appropriate software Repositories or 'repos'
The easies way is really to create an iso of the software necessary and use the iso files as the url for the repository. To add the repositories to the zypper configuration of the sles12sp3 server, we can use the either the 'yast' command or the "zypper ar ..." command. I have found it easier to use yast just because it avoids typos and such. 

Sles12sp3 comes with MariaDB, which is the prefered databse solution in a SLES deployment. However devstack is usually deployed onto a mysql database (the default).  In order to not confuse the subject we'll split the task into two different sections. 
- One: deploy devstack on sles12sp3 using a mysql backend
- Two: deploy devstack on sles12sp3 using a MariaDB backend

