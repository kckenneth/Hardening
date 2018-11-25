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




