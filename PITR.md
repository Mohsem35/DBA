PostgreSQL PITR on Ubuntu Easy Step-by-Step Guide

### 1. PITR(Point In TIme Recovery)

**Q. Why do we need PITR(Point In Time Recovery)?**

Before proceeding further, let us understand what could force us to perform a PITR.

1. Someone has accidentally dropped or truncated a table.
2. A failed deployment has made changes to the database that are difficult to reverse.
3. You accidentally deleted or modified a lot of data, and as a consequence, you cannot run your applications.

### 2. Getting Ready
2. 1 Prerequisite Steps
1. Enable sudo access to postgres:
For CentOS:
$ sudo vim /etc/sudoers
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL
postgres ALL=(ALL) ALL
For Ubuntu:
$ sudo usermod -a -G sudo postgres
2. Disable selinux or firewall:
For CentOS:
sudo su
sudo vim /etc/selinux/config
SELINUX=disabled
sudo reboot
2. 2 Primary Node Configuration
Step-1: Create archive directory in primary node where we could archive our WAL file
$ mkdir -p /var/lib/postgresql/14/pg14_archive
Step-2: Configure the postgresql.conf file for incremental backup and
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /var/lib/postgresql/14/pg14_archive/%f && cp %p
/var/lib/postgresql/14/pg14_archive/%f'
Meanings of the archive_command line:
%p=path to WAL file
%f=WAL file name
Step-3: Restart the postgresql
$ sudo systemctl restart postgresql
Force switch of WAL from psql: select pg_switch_wal();
Step-4: Create backup directory
$ mkdir -p /var/lib/postgresql/14/pg14_backup
Step-5: Run pg_basebackup utility as a postgres user
pg_basebackup -D /var/lib/postgresql/14/pg14_backup
Step-6: Stop the cluster and transfer all files to the main cluster and archive directory.
$ sudo systemctl stop postgresql@14.m
$ mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main_old
$ mkdir -p /var/lib/postgresql/14/main
$ cp -a /var/lib/postgresql/14/pg14_backup/. /var/lib/postgresql/14/main
$ chown postgres:postgres /var/lib/postgresql/14/main
$ chmod 700 /var/lib/postgresql/14/main
$ cd /var/lib/postgresql/14/main/pg_wal
postgres@ibprod-local:~/14/main/pg_wal$ rm -rf *
$ cp -a /var/lib/postgresql/14/main_old/pg_wal/. /var/lib/postgresql/14/main/pg_wal
$ cp /var/lib/postgresql/14/pg14_archive /var/lib/postgresql/14/main/pg_wal
or
postgres@ibprod-local:~/14/pg14_archive$ cp * /var/lib/postgresql/14/main/pg_wal
2.3 Standby Node Steps
Step-1: Compress the main cluster along with the archive directory from the primary node.
Here, 14 is the directory which will be compressed here
$ tar -cvzf /var/lib/postgresql/PITR14backup.tar.gz 14
Step-2: Copy the compress file from primary to standby node
$ scp /var/lib/postgresql/PITR14backup.tar.gz hostname@ip_address:~
Step-3: Stop the postgresql cluster and unzip the compressed file as a postgres user
$ sudo systemctl stop postgresql
$ sudo gunzip /home/ubuntu/PITR14backup.tar.gz
$ sudo tar -xvf /home/ubuntu/PITR14backup.tar
Step-4: Now we will destroy data from postgresql main cluster directory and copy the backup data into
postgres data directory. Execute following command:
$ rm -r /var/lib/postgresql/14/main/* # for postgresql 14
$ cp -a /home/ubuntu/14/* /var/lib/postgresql/14/
Step-5: Before starting postgresql, prepare postgresql to start in recovery mode. create
recovery.signal configuration file in postgres data directory of standby server as a postgres user.
$ touch /var/lib/postgresql/14/main/recovery.signal
$ ls -lh /var/lib/postgresql/14/main
-rw-r--r-- 1 postgres postgres 0 Aug 7 14:07 recovery.signal
Step-5: Configure the postgresql.conf file of the standby node
restore_command = 'cp /var/lib/postgresql/14/main/pg14_archive/%f %p'
Step-6: Start the postgresql
$ sudo systemctl start postgresql@14-main.service
3. Check If PITR Works
Check the log file
$ tail -100 /var/log/postgresql/postgresql-14-main.log
You should be able to watch the following line in the log
consistent recovery state reached at 0/66000138
4. There’s More
We can use PITR as our DRS(Disaster Recovery Solution) by setting recovery_target_time in
postgresql.conf file.
recovery_target_time = '2019-10-21 12:00:00'
