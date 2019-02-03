From: https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3
# Red Hat Ceph Storage 3 Administration Guide

### CHAPTER 1. CEPH ADMINISTRATION OVERVIEW
A Red Hat Ceph Storage cluster is the foundation for all Ceph deployments. After deploying a Red Hat Ceph Storage cluster, there are administrative operations for keeping a Red Hat Ceph Storage cluster healthy and performing optimally.

The Red Hat Ceph Storage Administration Guide helps storage administrators to perform such tasks as: How do I check the health of my Red Hat Ceph Storage cluster?

* How do I start and stop the Red Hat Ceph Storage cluster services?
* How do I add or remove an OSD from a running Red Hat Ceph Storage cluster?
* How do I manage user authentication and access controls to the objects stored in a Red Hat Ceph Storage cluster?
* I want to understand how to use overrides with a Red Hat Ceph Storage cluster.
* I want to monitor the performance of the Red Hat Ceph Storage cluster.

A basic Ceph Storage Cluster consist of two types of daemons:
* A Ceph Object Storage Device (OSD) stores data as objects within placement groups assigned to the OSD
* A Ceph Monitor maintains a master copy of the cluster map
* A production system will have three or more Ceph Monitors for high availability and typically a minimum of 50 OSDs for acceptable load balancing, data re-balancing and data recovery.

### CHAPTER 2. UNDERSTANDING PROCESS MANAGEMENT FOR CEPH

As a storage administrator, you can manipulate the Ceph daemons in various ways. Manipulating these daemons allows you to start, stop and restart all of the Ceph services as needed.
* Starting, Stopping, and Restarting All Ceph Daemons
* Starting, Stopping, Restarting a Ceph Daemon by Type
* Starting, Stopping, Restarting a Ceph Daemon by Instance

STARTING, STOPPING, AND RESTARTING ALL THE CEPH DAEMONS
###### On Ceph Cluster nodes
`[root@admin ~]# systemctl <start/stop/restart> ceph.target`

STARTING, STOPPING, AND RESTARTING THE CEPH DAEMONS BY TYPE
###### On Ceph Monitor nodes
`[root@mon ~]# systemctl <start/stop/restart> ceph-mon.target`
###### On Ceph OSD nodes
`[root@osd ~]# systemctl <start/stop/restart> ceph-osd.target`
###### On Ceph Object Gateway Nodes
`[root@rgw ~]# systemctl <start/stop/restat> ceph-radosgw.target`

STARTING, STOPPING, AND RESTARTING A CEPH DAEMONS BY INSTANCE
##### On a Ceph Monitor node:
`[root@mon ~]# systemctl <start/stop/restart> ceph-mon@$MONITOR_HOST_NAME`
##### Replace $MONITOR_HOST_NAME with the name of the Ceph Monitor node.
###### On a Ceph OSD node:
`[root@osd ~]# systemctl <start/stop/restart> ceph-osd@OSD_NUMBER`
##### Replace $OSD_NUMBER with the ID number of the Ceph OSD
###### On a Ceph Object Gateway node:
`[root@rgw ~]# systemctl <start/stop/restart> ceph-radosgw@rgw.$OBJ_GATEWAY_HOST_NAME`
##### Replace $OBJ_GATEWAY_HOST_NAME with the name of the Ceph Object Gateway node.

### CHAPTER 3. MONITORING

### CHAPTER 4. OVERRIDES
set or unset OSD flag
```
# ceph osd set <flag>
# ceph osd unset <flag>
```

USE CASES
* noin: Commonly used with noout to address flapping OSDs.
* noout: If the mon osd report timeout is exceeded and an OSD has not reported to the monitor, the OSD will get marked out. If this happens erroneously, you can set noout to prevent the OSD(s) from getting marked out while you troubleshoot the issue.
* noup: Commonly used with nodown to address flapping OSDs.
* nodown: Networking issues may interrupt Ceph 'heartbeat' processes, and an OSD may be upbut still get marked down. You can set nodown to prevent OSDs from getting marked down while troubleshooting the issue.
* full: If a cluster is reaching its full_ratio, you can pre-emptively set the cluster to full and expand capacity. NOTE: Setting the cluster to full will prevent write operations.
* pause: If you need to troubleshoot a running Ceph cluster without clients reading and writing data, you can set the cluster to pause to prevent client operations.
* nobackfill: If you need to take an OSD or node down temporarily, (e.g., upgrading daemons), you can set nobackfill so that Ceph will not backfill while the OSD(s) is down.
* norecover: If you need to replace an OSD disk and don’t want the PGs to recover to another OSD while you are hotswapping disks, you can set norecover to prevent the other OSDs from copying a new set of PGs to other OSDs.
* noscrub and nodeep-scrubb: If you want to prevent scrubbing (e.g., to reduce overhead during high loads, recovery, backfilling, rebalancing, etc.), you can set noscrub and/ornodeep-scrub to prevent the cluster from scrubbing OSDs.
* notieragent: If you want to stop the tier agent process from finding cold objects to flush to the backing storage tier, you may set notieragent.

### CHAPTER 5. USER MANAGEMENT
#### 5.2. MANAGEMENT USERS
##### 5.2.1. List USERS
>$ sudo ceph auth List
##### 5.2.2.
>$ sudo ceph auth get <TYPE.ID>
>$ sudo ceph auth get client.admin

** option -o <file_name>, output to a file **

** auth export is idntical to auth get **
##### 5.2.3. Add a user
```
# ceph auth add client.john mon 'allow r' osd 'allow rw pool=liverpool' # ceph auth get-or-create client.paul mon 'allow r' osd 'allow rw pool=liverpool'
# ceph auth get-or-create client.george mon 'allow r' osd 'allow rw pool=liverpool' -o george.keyring
# ceph auth get-or-create-key client.ringo mon 'allow r' osd 'allow rw pool=liverpool' -o ringo.key
```
##### 5.2.4. Modify user Capabilities
###### Syntax
```
# ceph auth caps <USERTYPE.USERID> <daemon> 'allow [r|w|x|*|...] [pool= <pool_name>] [namespace=<namespace_name>]'
```
###### Example
```
# ceph auth caps client.john mon 'allow r' osd 'allow rw pool=liverpool'
# ceph auth caps client.paul mon 'allow rw' osd 'allow rwx pool=liverpool'
# ceph auth caps client.brian-manager mon 'allow *' osd 'allow *'
```
###### To remove a capability
```
# ceph auth caps client.ringo mon ' ' osd ' '
```
##### 5.2.5. Delete a user
```
# ceph auth del {TYPE}.{ID}
```
##### 5.2.6. Print a User's key
To print a user's authentication key:
```
# ceph auth print-key <type>.<ID>
```
Print a user's key is useful when you need to populate client software with a user's key.
```
# mount -t ceph <hostname>:/<mount_point> -o name=client.user,secret=`ceph auth print-key client.user`
```
##### 5.2.7. Import a User
```
# ceph auth import -i </path/to/keying>
```

#### 5.3. KEYRING MANAGEMENT
When you access Ceph by using a Ceph client, the Ceph client will look for a local keyring. Ceph presets the **keyring** setting with the folowing four keyring names by default so you don't have to set them in the Ceph configuration file unless you want to override the defaults, which is not recommended:
* /etc/ceph/$cluster.$name.keyring
* /etc/ceph/$cluster.keyring
* /etc/ceph/keyring
* /etc/ceph/keyring.bin

##### 5.3.1 Create a keying
When you use the procedures in the Managing Users_ section to create users, you need to provide user keys to the Ceph client(s) so that the Ceph client can retrieve the key for the specified user and authenticate with the Ceph Storage Cluster. Ceph Clients access keyrings to lookup a user name and retrieve the user’s key.
