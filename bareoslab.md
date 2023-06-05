# INDEX
- [1. MariaDB backups and restore lab with Bareos](#1-mariadb-backups-and-restore-lab-with-bareos)
  - [1.1. Testing MariaDB backups](#11-testing-mariadb-backups)
  - [1.2. Preparaing the restore of MariaDB full backup](#12-preparaing-the-restore-of-mariadb-full-backup)
  - [1.3. Testing the restore of MariaDB full backups](#13-testing-the-restore-of-mariadb-full-backups)
  - [1.4. Preparing the restore of MariaDB incremental backup](#14-preparing-the-restore-of-mariadb-incremental-backup)
  - [1.5. Testing the restore of MariaDB incremental backup inc1](#15-testing-the-restore-of-MariaDB-incremental-backup-inc1)

# 1. MariaDB backups and restore lab with Bareos

This document has a proceedure to follow the backups and restore tests for MariaDB on Bareos. It can be repoduced following the detailed steps on the document and will prooves that Bareos can backup a MariaDB Galera cluster.

The following laboratory was made to be as close as possible of a real case:

![Network diagram](https://github.com/Franco-Sparrow/franco-repos/blob/master/WhatsApp%20Image%202023-04-17%20at%202.43.57%20PM.jpeg?raw=true)

## 1.1. Testing MariaDB backups

- Galera2 is the primary node (backup allways after Galera1 and Galera3).
- The hosts `galeradb1`, `galeradb2` and `galeradb3` are enabled in Zabbix.
- Backup FULL for Galera1: `Virtalus-Galera1-MariaDB-FULL-2023-5-2-12:22:6-Vol0`.
- Backup FULL for Galera3: `Virtalus-Galera3-MariaDB-FULL-2023-5-2-12:22:46-Vol0`.
- Backup FULL for Galera2: `Virtalus-Galera2-MariaDB-FULL-2023-5-2-12:23:21-Vol0`.
- The hosts `galeradb2` and `galeradb2` are enabled while `galeradb1` is disabled in Zabbix.
- Backup INC1 for Galera1: `Virtalus-Galera1-MariaDB-INCREMENTAL-2023-5-2-12:37:11-Vol0`.
- Backup INC1 for Galera3: `Virtalus-Galera3-MariaDB-INCREMENTAL-2023-5-2-12:38:25-Vol0`.
- Backup INC1 for Galera2: `Virtalus-Galera2-MariaDB-INCREMENTAL-2023-5-2-12:39:30-Vol0`.

## 1.2. Preparaing the restore of MariaDB full backup

> **NOTE** <br />
> The following steps should be done on each one of the Galera nodes.
>

Restore the FULL backup from Bareos WebUI, specifying the directory `/tmp/bareos-restores/MariaDB/full_not_ready`.

> **NOTE** <br />
>
> This is not mandatory, this steps are only for the testing the restores.
> 
> Make a copy of the restored files:
> ```bash
> rm -rf /tmp/bareos-restores/MariaDB/full_ready && \
> mkdir -p /tmp/bareos-restores/MariaDB/full_ready && \
> cp -r /tmp/bareos-restores/MariaDB/full_not_ready/* /tmp/bareos-restores/MariaDB/full_ready/
> ```
>

Prepare the backup, for example, for Galera1 it would will be as follow:

```
mariabackup --prepare \
  --target-dir=</path/to/full/backup>
```

```bash
mariabackup --prepare --target-dir=/tmp/bareos-restores/MariaDB/full_ready/_mariabackup/286/00000000000000000000_00000000002643581203_0000000280
```

The output is as follow:

```bash
mariabackup based on MariaDB server 10.6.12-MariaDB debian-linux-gnu (x86_64)
[00] 2023-06-05 07:12:22 cd to /tmp/bareos-restores/MariaDB/full_ready/_mariabackup/286/00000000000000000000_00000000002643581203_0000000280/
[00] 2023-06-05 07:12:22 open files limit requested 0, set to 1024
[00] 2023-06-05 07:12:22 This target seems to be not prepared yet.
[00] 2023-06-05 07:12:22 mariabackup: using the following InnoDB configuration for recovery:
[00] 2023-06-05 07:12:22 innodb_data_home_dir = .
[00] 2023-06-05 07:12:22 innodb_data_file_path = ibdata1:12M;ibdata2:12M:autoextend
[00] 2023-06-05 07:12:22 innodb_log_group_home_dir = .
[00] 2023-06-05 07:12:22 InnoDB: Using Linux native AIO
[00] 2023-06-05 07:12:22 Starting InnoDB instance for recovery.
[00] 2023-06-05 07:12:22 mariabackup: Using 104857600 bytes for buffer pool (set by --use-memory parameter)
2023-06-05  7:12:22 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2023-06-05  7:12:22 0 [Note] InnoDB: Number of pools: 1
2023-06-05  7:12:22 0 [Note] InnoDB: Using generic crc32 instructions
2023-06-05  7:12:22 0 [Note] InnoDB: Using Linux native AIO
2023-06-05  7:12:22 0 [Note] InnoDB: Initializing buffer pool, total size = 104857600, chunk size = 104857600
2023-06-05  7:12:22 0 [Note] InnoDB: Completed initialization of buffer pool
2023-06-05  7:12:22 0 [Note] InnoDB: Starting crash recovery from checkpoint LSN=2643581203,2649216325
2023-06-05  7:12:22 0 [Note] InnoDB: Starting final batch to recover 1884 pages from redo log.
[00] 2023-06-05 07:12:23 Last binlog file , position 0
[00] 2023-06-05 07:12:23 completed OK!
```

*Now the FULL backup is ready to be restored in MariaDB!!!*

> **NOTE** <br />
>
> Do not continue until the others galera nodes restore their full backups too. <br />
>
> - For galeradb2: `288/00000000000000000000_00000000002642910854_0000000281`
> - For galeradb3: `287/00000000000000000000_00000000002643923766_0000000282`
>

## 1.3. Testing the restore of MariaDB full backups

If you have finised with the other galera nodes, stop mariadb.service and delete all files from mariadb data dir:

```bash
systemctl stop mariadb.service && \
rm -rf /var/lib/mysql/ && \
mkdir -p /var/lib/mysql/ && \
chown mysql:mysql /var/lib/mysql/ && \
chmod 755 /var/lib/mysql/
```

Make sure that mariadb is stopped:

```bash
systemctl status mariadb.service
```

The output should look as follow:

```bash
● mariadb.service - MariaDB 10.6.12 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/mariadb.service.d
             └─migrated-from-my.cnf-settings.conf
     Active: inactive (dead) since Tue 2023-05-02 13:11:24 PDT; 1h 23min ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 103634 ExecStart=/usr/sbin/mariadbd $MYSQLD_OPTS $_WSREP_NEW_CLUSTER $_WSREP_START_POSITION (code=exited, status=0/SUCCESS)
   Main PID: 103634 (code=exited, status=0/SUCCESS)
     Status: "MariaDB server is down"

May 02 05:44:36 galeradb2 systemd[1]: Starting MariaDB 10.6.12 database server...
May 02 05:44:37 galeradb2 sh[103474]: WSREP: Recovered position 719d622b-cddc-11ed-9c6c-871919c8a588:1636358
May 02 05:44:38 galeradb2 systemd[1]: Started MariaDB 10.6.12 database server.
May 02 13:11:23 galeradb2 systemd[1]: Stopping MariaDB 10.6.12 database server...
May 02 13:11:24 galeradb2 systemd[1]: mariadb.service: Succeeded.
May 02 13:11:24 galeradb2 systemd[1]: Stopped MariaDB 10.6.12 database server.
```

> **NOTE** <br />
> If your galera is behind an LB (ie, haproxy), stop the service: <br />
>
> ```
> systemctl stop haproxy.service
> ```

Finally, restore it into MariaDB:

```
mariabackup --copy-back \
  --target-dir=</path/to/full/backup>
```

```bash
mariabackup --copy-back --target-dir=/tmp/bareos-restores/MariaDB/full_ready/_mariabackup/286/00000000000000000000_00000000002643581203_0000000280
```

The output is as follow:

```bash
# [...]
[01] 2023-05-02 14:09:41         ...done
[00] 2023-05-02 14:09:41 completed OK!
```

Restore the ownership on the restored files:

```bash
chown -R mysql:mysql /var/lib/mysql/
```

*Once you have done the previous steps for all galera nodes, proceed to start the mariadb.service!!!*

Start mariadb on last primary node from Galera (on Galera2):

```bash
galera_new_cluster
```

Start mariadb on the other nodes (Galera1 and Galera3):

```bash
systemctl start mariadb.service
```

Verify the state of the Galera cluster:

```bash
mysql -uroot -e "SELECT * FROM information_schema.global_status WHERE variable_name IN ('WSREP_CLUSTER_STATUS','WSREP_LOCAL_STATE_COMMENT','WSREP_CLUSTER_SIZE','WSREP_EVS_DELAYED','WSREP_READY');"
```

```bash
+---------------------------+----------------+
| VARIABLE_NAME             | VARIABLE_VALUE |
+---------------------------+----------------+
| WSREP_LOCAL_STATE_COMMENT | Synced         |
| WSREP_EVS_DELAYED         |                |
| WSREP_CLUSTER_SIZE        | 3              |
| WSREP_CLUSTER_STATUS      | Primary        |
| WSREP_READY               | ON             |
+---------------------------+----------------+
```

> **NOTE** <br />
> If your galera is behind an LB (ie, haproxy), start the service: <br />
>
> ```bash
> systemctl start haproxy.service
> ```
>

Get access to Zabbix frontend, **all galera hosts should appear enabled!!!**.

![Galera hosts status after restored the Full backup](https://github.com/Franco-Sparrow/franco-repos/assets/56793015/ccea5c18-aaaa-48d0-bdea-bac94e341fe0)

## 1.4. Preparing the restore of MariaDB incremental backup

> **NOTE** <br />
> The following steps should be done on each one of the galera nodes.
>

Restore the inc1 backup, specifying the directory `/tmp/bareos-restores/MariaDB/inc1_not_ready`.

> **NOTE** <br /
>
> This is not mandatory, this steps are only for the testing the restores.
> 
> Make a copy of the restored files:
> ```bash
> rm -rf /tmp/bareos-restores/MariaDB/full_and_inc1_ready && \
> mkdir -p /tmp/bareos-restores/MariaDB/full_and_inc1_ready && \
> cp -r /tmp/bareos-restores/MariaDB/full_ready/* /tmp/bareos-restores/MariaDB/full_and_inc1_ready/
> ```
> 

Prepare the backup, for example, for Galera1 it would will be as follow:

```
mariabackup --prepare \
  --target-dir=</path/to/full/backup> \
  --incremental-dir=</path/to/inc1/backup>
```

```bash
mariabackup --prepare --target-dir=/tmp/bareos-restores/MariaDB/full_and_inc1_ready/_mariabackup/286/00000000000000000000_00000000002643581203_0000000280 --incremental-dir=/tmp/bareos-restores/MariaDB/inc1_not_ready/_mariabackup/172/00000000001093163231_00000000001093163231_0000000158
```

```bash
mariabackup based on MariaDB server 10.6.12-MariaDB debian-linux-gnu (x86_64)
# [...]
[00] 2023-05-02 15:08:35         ...done
[00] 2023-05-02 15:08:35 completed OK!
```

*Now the resulting backup has FULL+INC1 and is ready to be restored in MariaDB!!!*

## 1.5. Testing the restore of MariaDB incremental backup inc1

If you have finised with the other galera nodes, stop mariadb.service and delete all files from mariadb data dir:

```bash
systemctl stop mariadb.service && \
rm -rf /var/lib/mysql/ && \
mkdir -p /var/lib/mysql/ && \
chown mysql:mysql /var/lib/mysql/ && \
chmod 755 /var/lib/mysql/
```

Make sure that mariadb is stopped:

```bash
systemctl status mariadb.service
```

The output should look as follow:

```bash
● mariadb.service - MariaDB 10.6.12 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/mariadb.service.d
             └─migrated-from-my.cnf-settings.conf
     Active: inactive (dead) since Tue 2023-05-02 15:10:51 PDT; 24s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 10109 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 10113 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 10119 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`cd /usr/bin/..; /usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$>
    Process: 10283 ExecStart=/usr/sbin/mariadbd $MYSQLD_OPTS $_WSREP_NEW_CLUSTER $_WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 10508 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 10510 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 10283 (code=exited, status=0/SUCCESS)
     Status: "MariaDB server is down"

May 02 14:42:52 galeradb1 rsyncd[10457]: sent 6356 bytes  received 247168243 bytes  total size 247085055
May 02 14:42:52 galeradb1 rsyncd[10474]: connect from galeradb2 (10.0.87.13)
May 02 14:42:52 galeradb1 rsyncd[10474]: rsync to rsync_sst/ from galeradb2 (10.0.87.13)
May 02 14:42:52 galeradb1 rsyncd[10474]: receiving file list
May 02 14:42:52 galeradb1 rsyncd[10474]: sent 48 bytes  received 195 bytes  total size 43
May 02 14:42:53 galeradb1 rsyncd[10419]: sent 0 bytes  received 0 bytes  total size 0
May 02 14:42:55 galeradb1 systemd[1]: Started MariaDB 10.6.12 database server.
May 02 15:10:50 galeradb1 systemd[1]: Stopping MariaDB 10.6.12 database server...
May 02 15:10:51 galeradb1 systemd[1]: mariadb.service: Succeeded.
May 02 15:10:51 galeradb1 systemd[1]: Stopped MariaDB 10.6.12 database server.
```

> **NOTE** <br />
> If your galera is behind an LB (ie, haproxy), stop the service: <br />
>
> ```
> systemctl stop haproxy.service
> ```

Finally, restore it into MariaDB:

```
mariabackup --copy-back \
  --target-dir=</path/to/full/backup>
```

```bash
mariabackup --copy-back --target-dir=/tmp/bareos-restores/MariaDB/full_and_inc1_ready/_mariabackup/167/00000000000000000000_00000000001093163231_0000000155
```

The output is as follow:

```bash
# [...]
[01] 2023-05-02 14:09:41         ...done
[00] 2023-05-02 14:09:41 completed OK!
```

Restore the ownership on the restored files:

```bash
chown -R mysql:mysql /var/lib/mysql/
```

*Once you have done the previous steps for all galera nodes, proceed to start the mariadb.service!!!*

Start mariadb on last primary node from Galera (**Galera2**):

```bash
galera_new_cluster
```

Start mariadb on the other nodes (Galera1 and Galera3):

```bash
systemctl start mariadb.service
```

Verify the state of the Galera cluster:

```bash
mysql -uroot -e "SELECT * FROM information_schema.global_status WHERE variable_name IN ('WSREP_CLUSTER_STATUS','WSREP_LOCAL_STATE_COMMENT','WSREP_CLUSTER_SIZE','WSREP_EVS_DELAYED','WSREP_READY');"
```

```bash
+---------------------------+----------------+
| VARIABLE_NAME             | VARIABLE_VALUE |
+---------------------------+----------------+
| WSREP_LOCAL_STATE_COMMENT | Synced         |
| WSREP_EVS_DELAYED         |                |
| WSREP_CLUSTER_SIZE        | 3              |
| WSREP_CLUSTER_STATUS      | Primary        |
| WSREP_READY               | ON             |
+---------------------------+----------------+
```

> **NOTE** <br />
> If your galera is behind an LB (ie, haproxy), start the service: <br />
>
> ```
> systemctl star haproxy.service
> ```
> 

Get access to Zabbix frontend, **the hosts `galeradb2` and `galeradb2` should be enabled while `galeradb1` should be disabled!!!**.

![Galera hosts status after restored the Incremental backup](https://github.com/Franco-Sparrow/franco-repos/assets/56793015/ea97b0a2-b8d8-49a6-a43a-2238128cf458)
