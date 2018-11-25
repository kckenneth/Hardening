# Hardening

This is a detailed walkthrough on how to harden your server. 

|Local (from) |   | Server (remote) |
|-----------|---|-------------------------------|
| /etc/hosts |  | /etc/ssh/sshd_config
| ~/.ssh/config |  | /etc/sudoers |

## VM provisioning 

Create 3 nodes with 4 CPU, 32G RAM, 100G disk with CentOS.

```
$ slcli vs create --datacenter=hou02 --hostname=spark1 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark2 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
$ slcli vs create --datacenter=hou02 --hostname=spark3 --domain=mids.com --billing=hourly --cpu=4 --memory=32768 --disk=100 --os=CENTOS_LATEST_64
```

Checking the servers provisioned

```
$ slcli vs list

:..........:..........:................:..............:............:........:
:    id    : hostname :   primary_ip   :  backend_ip  : datacenter : action :
:..........:..........:................:..............:............:........:
: 65711055 :  spark1  : 184.173.57.205 : 10.76.54.135 :   hou02    :   -    :
: 65711727 :  spark2  : 184.173.57.202 : 10.76.54.136 :   hou02    :   -    :
: 65711737 :  spark3  : 184.173.57.203 : 10.76.54.151 :   hou02    :   -    :
:..........:..........:................:..............:............:........:
```

You need to know the password for the first step by calling `slcli vs credentials 65711055` and other VMs as well. 

## Procedures (Local)

We will login to our remote servers for the first time with the passwords. After this step, we try not to use the password, which requires us to setup the ssh `id_rsa` private and public keys. Since hardening the server also requires try not to use the root login, we will then create a user account in each VM we provisioned. 

1. create a user in each server 
2. allow the user adminstrative power  
3. allow the username login  
4. change the listening port default 22 to your choice
5. block the password authentication and rootlogin permanently 

From your local computer (laptop), create ssh keys. If done, skip this step. 
```
$ ssh-keygen -f ~/.ssh/id_rsa -b 2048 -t rsa 
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
$ chmod 600 ~/.ssh/authorized_keys
```
Open 3 terminals. Do in each terminal, copy the `id_rsa.pub` to remote servers. 
```
$ ssh-copy-id root@184.173.57.205             # Will require the password, enter the password
$ ssh root@184.173.57.205                     # You no longer need the password in this step
```

## Configuring in the remote servers (All of the servers)
The following steps need to be done in all of the servers. You need to execute in each of the server separately. You can do so by opening three terminals. The walkthrough below only shows for one server. 

## 1. Create a user
```
$ adduser kenneth
$ passwd kenneth                              # set the password for the user kenneth
```

## 2. Allow adminstrator power to the user
```
$ vi /etc/sudoers

root	ALL=(ALL) 	ALL                         # default line
kenneth ALL = NOPASSWD: ALL                   # format username All = NOPASSWD: ALL
```

## 3. 

AllowUsers root kenneth
Port 4000
PasswordAuthentication no





