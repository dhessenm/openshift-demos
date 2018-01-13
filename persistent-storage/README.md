# OpenShift Demo MySQL database with persistent storage
## Configure NFS Server
hosts: ns1
Create  NFS share directories

```
mkdir -p /home/data/pv0001
mkdir -p /home/data/pv0002
chmod -R 777 /home/data/
```

Create file /etc/exports:

```
/home/data/pv0001 *(rw,sync)
/home/data/pv0002 *(rw,sync)
```

enable and start  NFS-Server, NFS shares

```
exportfs -a
systemctl enable nfs
systemctl nfs start
showmount -e
```

## Configure OpenShift nodes
hosts: osmaster1 und osnode1
Make sure, that NFS works on OpenShift nodes with NFS sever, test it:

```
mount -t nfs -o hard ns1.joker.db.de:/home/data/pv0009 /tmp/test1
umount (-f -l) /tmp/test1
```

Configure SELinux (pods can use/access  NFS volumes on host)

```
setsebool -P virt_use_nfs 1
```

## Create persistent volume

Create a persistent volume file, e.g. pv0002.yml:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0002
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: ns1.joker.db.de
    path: /home/data/pv0002
```

Make persistent storage available in OpenShift:

```
oc create -f pv0002.yml
```

## Erzeuge MySQL DB mit persistent storage
Create in example OpenShift namespace (e.g. persistent-storage-demo) a persistent volume claim (pvc). 
Or create a MySQL Database directly from Template 'MySQL persistent' (from OpenShift namespace 'openshift'). 

```
Username: mysql
Password: 12345678
Database Name: sampledb
Connection URL: mysql://mysql:3306/
```

## Check data persistence in MySQL database

```
oc get pods
oc get pvc
oc port-forward -p <pod name> <port>:<port>
```

open a new (second) terminal window:

```
ns1: oc port-forward -p mysql-1-s46mf 3306:3306
```

Test database

```
mysql -h 127.0.0.1 -P 3306 -uroot -p12345678
use sampledb
create table users (user_id int not null auto_increment, username varchar(200), PRIMARY KEY(user_id));
desc users;
insert into users values (null, 'Dirk');
insert into users values (null, 'Holger');
select * from users;
```

Delete MySQL pod (OpenShift creates immediately a new one)

```
oc delete pod  mysql-1-s46mf
```

Is new pod available?

```
oc get pods
```

forward to new pod:

```
oc port-forward -p mysql-1-6wx2x 3306:3306
```

check, if table 'users' exists and contains expected data:

```
use sampledb;
show tables;
select * from users;
```

# References
Doku: https://docs.openshift.com/container-platform/3.6/using_images/db_images/mysql.html
Gute Demo: https://blog.openshift.com/openshift-demo-part-12-using-persistent-storage/
