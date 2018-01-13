# OpenShift Demo MySQL Datenbank mit Persistent Storage
Doku: https://docs.openshift.com/container-platform/3.6/using_images/db_images/mysql.html

Gute Demo: https://blog.openshift.com/openshift-demo-part-12-using-persistent-storage/

## Konfiguriere NFS Server
hosts: ns1
Anlegen von NFS Share Verzeichnissen

```
mkdir -p /home/data/pv0001
mkdir -p /home/data/pv0002
chmod -R 777 /home/data/
```

Datei /etc/exports:

```
/home/data/pv0001 *(rw,sync)
/home/data/pv0002 *(rw,sync)
```

Aktiviere NFS-Server

```
exportfs -a
systemctl enable nfs
systemctl nfs start
showmount -e
```

## Konfiguriere OpenShift Server
hosts: osmaster1 und osnode1
Test

```
mount -t nfs -o hard ns1.joker.db.de:/home/data/pv0009 /tmp/test1
umount (-f -l) /tmp/test1
```

SELinux konfigurieren, damit Pods auf NFS Volumes zugreifen können:

```
setsebool -P virt_use_nfs 1
```

## Erzeuge Persistent Volume

Erzeuge PV Datei, Beispiel pv0002.yml:

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

Mache persistent storage in OpenShift verfügbar:

```
oc create -f pv0002.yml
```

## Erzeuge MySQL DB mit persistent storage
Erzeuge im Projekt ein Persistent Volume Claim (PVC). Dieses ist direkt im OpenShift Templete Mysql Persistent (Namespase OpenShift) enthalten. Führe das Template im eigenen Namespace aus. Wähle und merke MySQL-Daten, z. B.:

```
Username: mysql
Password: 12345678
Database Name: sampledb
Connection URL: mysql://mysql:3306/
```

## Checke Datenpersistenz in derv Datenbank

```
oc get pods
oc get pvc
oc port-forward -p <pod name> <port>:<port>
```

in weiterem Fenster:

```
ns1: oc port-forward -p mysql-1-s46mf 3306:3306
```

Test Database

```
mysql -h 127.0.0.1 -P 3306 -uroot -p12345678
use sampledb
create table users (user_id int not null auto_increment, username varchar(200), PRIMARY KEY(user_id));
desc users;
insert into users values (null, 'Dirk');
insert into users values (null, 'Holger');
select * from users;
```

Lösche den MySQL-Pod (es wird automatisch ein neuer Pod gestartet)

```
oc delete pod  mysql-1-s46mf
```

neuer pod da?

```
oc get pods
```

forward to new pod:

```
oc port-forward -p mysql-1-6wx2x 3306:3306
```

check, ob Tabelle users mir Daten noch vorhanden ist.

```
use sampledb;
show tables;
select * from users;
```

