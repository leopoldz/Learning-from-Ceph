#### 管理功能测试
集群配置文件:
/etc/ceph/ceph.conf
```
[global]
fsid = 868da540-60fa-4d05-87ee-2f9097123e6a
mon_initial_members = cfphdd001
mon host = 10.18.18.1:6789, 10.18.18.2:6789, 10.18.18.3:6789
mon_allow_pool_delete = true
# mon_host = 10.18.18.1

public_network = 10.18.18.0/24
cluster_network = 10.18.19.0/24

auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

# osd journal size = {n}
osd pool default size = 2  # Write an object n times.
# osd pool default min size = {n} # Allow writing n copy in a degraded state.
osd pool default pg num = 100
osd pool default pgp num = 100

[mon]
mgr_initial_modules = dashboard

[client]
```
###### 推送配置文件
>$ ceph-deploy --overwrite-conf config push cfphdd00{1..3} cfpssd10{1..3} cfadmin
###### nobackfill 打开
>$ sudo ceph osd set nobackfill
###### 重新启动 OSDs
>$ ansible -b OSDs -a 'systemctl restart ceph-osd.target'
###### nobackfill 关闭
>$ sudo ceph osd unset nobackfill
###### 重新启动 MONs
>$ ansible -b MONs -a 'systemctl restart ceph-mon.target'
###### delete mytest7 pool
>$ sudo ceph osd pool rm mytest7 mytest7 --yes-i-really-really-mean-it
###### 检查健康状态
>$ sudo ceph -s
###### Set pg_num
>$ sudo ceph osd lspools | awk '{print $2}' | while read a; do echo echo -n $a =\; sudo ceph osd pool set $a pg_num 100; done | sh
###### Set pgp_num
>$ sudo ceph osd lspools | awk '{print $2}' | while read a; do echo echo -n $a =\; sudo ceph osd pool set $a pgp_num 100; done | sh
###### Set pool crush_rule to hot (device class: ssd)
>$ sudo ceph osd lspools | awk '{print $2}' | while read a; do echo $a; sudo ceph osd pool set $a crush_rule hot; done
###### Set pool size from 3 to 2(replicas)
>$ sudo ceph osd lspools | awk '{print $2}' | while read a; do echo $a; sudo ceph osd pool set $a size 2; done
###### Add a new pool
>$ sudo ceph osd pool create mytest2 100 100
###### Set crush_rule to pool
>$ sudo ceph osd pool set mytest2 crush_rule hot
###### Set new pg_num
>$ sudo ceph osd pool set mytest2 pg_num 300
###### Set new pgp_num
>$ sudo ceph osd pool set mytest2 pgp_num 300

###### Get Health_ok
>$ sudo ceph -s
