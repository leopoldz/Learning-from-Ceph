From Red Hat Ceph Storage 3 Administration Guide

CHAPTER 1. CEPH ADMINISTRATION OVERVIEW
A Red Hat Ceph Storage cluster is the foundation for all Ceph deployments. After deploying a Red Hat Ceph Storage cluster, there are administrative operations for keeping a Red Hat Ceph Storage cluster healthy and performing optimally.
The Red Hat Ceph Storage Administration Guide helps storage administrators to perform such tasks as: How do I check the health of my Red Hat Ceph Storage cluster?
How do I start and stop the Red Hat Ceph Storage cluster services?
How do I add or remove an OSD from a running Red Hat Ceph Storage cluster?
How do I manage user authentication and access controls to the objects stored in a Red Hat Ceph Storage cluster?
I want to understand how to use overrides with a Red Hat Ceph Storage cluster.
I want to monitor the performance of the Red Hat Ceph Storage cluster.
A basic Ceph Storage Cluster consist of two types of daemons:
A Ceph Object Storage Device (OSD) stores data as objects within placement groups assigned to the OSD
A Ceph Monitor maintains a master copy of the cluster map
A production system will have three or more Ceph Monitors for high availability and typically a minimum of 50 OSDs for acceptable load balancing, data re-balancing and data recovery.

CHAPTER 2. UNDERSTANDING PROCESS MANAGEMENT FOR CEPH

As a storage administrator, you can manipulate the Ceph daemons in various ways. Manipulating these daemons allows you to start, stop and restart all of the Ceph services as needed.
Starting, Stopping, and Restarting All Ceph Daemons
Starting, Stopping, Restarting a Ceph Daemon by Type
Starting, Stopping, Restarting a Ceph Daemon by Instance

STARTING, STOPPING, AND RESTARTING ALL THE CEPH DAEMONS
#### On Ceph Cluster nodes
[root@admin ~]# systemctl <start/stop/restart> ceph.target
STARTING, STOPPING, AND RESTARTING THE CEPH DAEMONS BY TYPE
#### On Ceph Monitor nodes
[root@mon ~]# systemctl <start/stop/restart> ceph-mon.target
#### On Ceph OSD nodes
[root@osd ~]# systemctl <start/stop/restart> ceph-osd.target
#### On Ceph Object Gateway Nodes
[root@rgw ~]# systemctl <start/stop/restat> ceph-radosgw.target
STARTING, STOPPING, AND RESTARTING A CEPH DAEMONS BY INSTANCE
#### On a Ceph Monitor node:
[root@mon ~]# systemctl <start/stop/restart> ceph-mon@$MONITOR_HOST_NAME
#### Replace $MONITOR_HOST_NAME with the name of the Ceph Monitor node. 
#### On a Ceph OSD node:
[root@osd ~]# systemctl <start/stop/restart> ceph-osd@OSD_NUMBER
#### Replace $OSD_NUMBER with the ID number of the Ceph OSD
#### On a Ceph Object Gateway node:
[root@rgw ~]# systemctl <start/stop/restart> ceph-radosgw@rgw.$OBJ_GATEWAY_HOST_NAME
#### Replace $OBJ_GATEWAY_HOST_NAME with the name of the Ceph Object Gateway node.
CHAPTER 3. MONITORING

CHAPTER 4. OVERRIDES
set or unset OSD flag
# ceph osd set <flag>
# ceph osd unset <flag>
USE CASES
noin: Commonly used with noout to address flapping OSDs.
noout: If the mon osd report timeout is exceeded and an OSD has not reported to the monitor, the OSD will get marked out. If this happens erroneously, you can set noout to prevent the OSD(s) from getting marked out while you troubleshoot the issue.
noup: Commonly used with nodown to address flapping OSDs.
nodown: Networking issues may interrupt Ceph 'heartbeat' processes, and an OSD may be upbut still get marked down. You can set nodown to prevent OSDs from getting marked down while troubleshooting the issue.
full: If a cluster is reaching its full_ratio, you can pre-emptively set the cluster to full and expand capacity. NOTE: Setting the cluster to full will prevent write operations.
pause: If you need to troubleshoot a running Ceph cluster without clients reading and writing data, you can set the cluster to pause to prevent client operations.
nobackfill: If you need to take an OSD or node down temporarily, (e.g., upgrading daemons), you can set nobackfill so that Ceph will not backfill while the OSD(s) is down.
norecover: If you need to replace an OSD disk and don’t want the PGs to recover to another OSD while you are hotswapping disks, you can set norecover to prevent the other OSDs from copying a new set of PGs to other OSDs.
noscrub and nodeep-scrubb: If you want to prevent scrubbing (e.g., to reduce overhead during high loads, recovery, backfilling, rebalancing, etc.), you can set noscrub and/ornodeep-scrub to prevent the cluster from scrubbing OSDs.
notieragent: If you want to stop the tier agent process from finding cold objects to flush to the backing storage tier, you may set notieragent.

CHAPTER 5. USER MANAGEMENT

