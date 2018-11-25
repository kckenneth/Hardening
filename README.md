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
5. block the password authentication and rootlogin permanently (Very last step until

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

## 3. and 4. Allow username login and change port
```
$ vi /etc/ssh/sshd_config

AllowUsers root kenneth                       # if not available, create your own
Port 4000                                     # default 22, if commented, decomment and change the port number
```
Everytime you make changes in `sshd_config`, to activate the changes, you need to restart the service. 
```
$ service sshd restart
```

## Working on remote servers as local  
The following steps are done from the perspective of the local user. Imagine you're now in the remote server 1, but you want to connect to another remote server2, the server1 becomes the local computer and server2 becomes the remote server. If you're in remote server2 and wants to connect to the server1, the server2 becomes your local computer and server1 become a remote server. Since we want each server communicate freely without password (Hadoop/Spark etc etc), we need to work on every server as `local` and `remote` aspect. 

Now that we have changed the port in every server from the default 22 to 4000, we need to specify the port when we log in. This is an optional step to show you. 
```
$ ssh -p 4000 root@IP
```
## Local user in Remote Server (All of the servers)
Since we will be using our username `kenneth`, you now need to login as the user `kenneth`. Do the following in each of the server. 
```
$ su - kenneth
[kenneth@spark1 ~]$
```
You're now in the server1 as the user `kenneth`. Everything you make changes will be under the username, not the root. This is important because if you're changing as a `root` admin and want to login to a remote server which is setup for the username `kenneth`, everything will break down. 

Setting up the DNS
```
$ vi /etc/hosts

127.0.0.1   localhost.localdomain localhost
10.76.54.135 spark1.mids.com spark1
10.76.54.136 spark2.mids.com spark2
10.76.54.151 spark3.mids.com spark3
```
Since we have change the port in all of our servers as root, this requires `anyone` connecting to the servers using the specified port. So now we need to setup the port in username `kenneth` account. 

```
$ vi ~/.ssh/config

Host spark1
   HostName spark1
   Port 4000
   User kenneth

Host spark2
   HostName spark2
   Port 4000
   User kenneth

Host spark3
   HostName spark3
   Port 4000
   User kenneth
   
$ chmod 700 ~/.ssh/config
```
The `User` is optional. Since we're setting up the user `kenneth`, we need to specify the user. If you're doing as a `root`, you don't need to add the `User kenneth` line. 

So far we haven't actually establish any connection between 3 of our remote servers. Since have already setup DNS and port, let's try and see if it works. 

### From remote server 1 (spark1)
We need to setup the `kenneth` ssh password `id_rsa` to establish between all of our servers. This is compulsory. 

```
$ ssh-keygen -f ~/.ssh/id_rsa -b 2048 -t rsa 
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
$ chmod 600 ~/.ssh/authorized_keys
$ ssh-copy-id spark2                          # Will require kenneth password in spark2, enter the password
$ ssh-copy-id spark3                          # Will require kenneth password in spark3, enter the password
$ scp ~/.ssh/* spark2:/home/kenneth/.ssh      # this will copy everything
$ scp ~/.ssh/* spark3:/home/kenneth/.ssh      
```
Now that we have established the user connection in all of our remote servers `AllowUsers`, we also established the `id_rsa` private and public keys in all of the servers, we also configured the port in every remote server `/etc/ssh/sshd_config` and specify which port to use when we login in from our local server `~/.ssh/config`, I think we're now good to check our passwordless connection between each server. First we need to make sure that even spark1 can ssh into its own spark1 without password. 
```
[kenneth@spark1 ~]$ ssh spark1                # if this asks ECDSA, enter yes
```
If this passes without you to enter the password, we now established the passwordless connection. Ok now we need to test all the servers. 
```
$ vi test.sh

#!/bin/bash

# Edit node list
nodes="spark1 spark2 spark3"

# Test ssh configuration
for i in $nodes
do for j in $nodes
 do echo -n "Testing ${i} to ${j}: "
 ssh  ${i} "ssh ${j} date"
 done
done
```
Check the connection
```
$ chmod 755 test.sh
$ ./test.sh

[kenneth@spark1 ~]$ ./test.sh
Testing spark1 to spark1: Sun Nov 25 14:27:53 CST 2018
Testing spark1 to spark2: Sun Nov 25 14:27:54 CST 2018
Testing spark1 to spark3: Sun Nov 25 14:27:55 CST 2018
Testing spark2 to spark1: Sun Nov 25 14:27:56 CST 2018
Testing spark2 to spark2: Sun Nov 25 14:27:57 CST 2018
Testing spark2 to spark3: Sun Nov 25 14:27:57 CST 2018
Testing spark3 to spark1: Sun Nov 25 14:27:58 CST 2018
Testing spark3 to spark2: Sun Nov 25 14:27:59 CST 2018
Testing spark3 to spark3: Sun Nov 25 14:28:00 CST 2018
```
If the script generated the above message, we're good to go!

Now that we have established the user login without requiring the password and using the different port, we can now block any of the root login and password authentication. 

## 5. Block password authentication and rootlogin
Since we're in `kenneth` user account in each server, we need to log out of the user to get back to `root`. This step is very important because if you do that, you will have a strong protection against the server attack. At the same time, you lose your `id_rsa` key, you'll be locked out of your own server and won't be able to connect. For me I only like to use `PasswordAuthentication no` because if in case, I need to change something and need to log back in as a root user, I don't want to be locked out. Anyway, I show how to block the root login as an option. 
```
$ vi /etc/ssh/sshd_config
PasswordAuthentication no                     # change yes to no
PermitRootLogin no                            # (Optional parameter to change) if commented, decomment and change yes to no

$ service sshd restart
```







