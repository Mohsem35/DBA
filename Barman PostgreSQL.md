Allows your company to implement **`disaster recovery solutions`** for PostgreSQL databases with high requirements of business continuity.


### Install PostgreSQL
```bash
# Create the file repository configuration:
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Update the package lists:
sudo apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql
```


### Create Barman User in PostgreSQL

```bash
$ sudo -u postgres psql
```
Run all the following commands as a `postgres` user
```sql
CREATE ROLE barman WITH LOGIN PASSWORD 'barman';
ALTER ROLE barman WITH superuser;

GRANT EXECUTE ON FUNCTION pg_start_backup(text, boolean, boolean) to barman;
GRANT EXECUTE ON FUNCTION pg_stop_backup() to barman;
GRANT EXECUTE ON FUNCTION pg_stop_backup(boolean, boolean) to barman;
GRANT EXECUTE ON FUNCTION pg_switch_wal() to barman;
GRANT EXECUTE ON FUNCTION pg_create_restore_point(text) to barman;

GRANT pg_read_all_settings TO barman;
GRANT pg_read_all_stats TO barman;
```

### Install Barman

```bash
sudo su
apt-get install barman -y

barman --version
3.5.0 Barman by EnterpriseDB (www.enterprisedb.com)

passwd barman
New password:barman
Retype new password:barman 
password updated successfully
```


### Configure SSH Key-based Authentication on Both Servers


#### Configure SSH Key-based Authentication on PostgreSQL Server

```
postgres@pghost:~$ ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa
postgres@pghost:~$ ssh-copy-id -i ~/.ssh/id_rsa.pub barman@172.16.6.5
postgres@pghost:~$ ssh barman@172.16.6.5 "chmod 600 ~/.ssh/authorized_keys"
```
#### Configure SSH Key-based Authentication on Barman Server
```
barman@bmhost:~$ ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa
barman@bmhost:~$ ssh-copy-id -i ~/.ssh/id_rsa.pub postgres@172.16.5.57
barman@bmhost:~$ ssh postgres@172.16.5.57 "chmod 600 ~/.ssh/authorized_keys"
```

### Configure PostgreSQL Server
```
vim /etc/postgresql/14/main/postgresql.conf
```
```
listen_addresses = '*'
wal_level = replica 
archive_mode = on
archive_command = 'test ! -f barman@172.16.6.5:/var/lib/barman/pghost/incoming/%f && rsync -a %p barman@172.16.6.5:/var/lib/barman/pghost/incoming/%f'
```

```
postgres=# show archive_command;

 					archive_command
-------------------------------------------------------------------------------------
test ! -f barman@test-machine01:/var/lib/barman/test-machine02/incoming/%f && rsync
-a %p barman@test-machine01:/var/lib/barman/test-machine02/incoming/%f
```
```
# vim /etc/postgresql/14/main/pg_hba.conf

host	all         	all         	172.16.6.5/32       	trust
host	replication 	all         	172.16.6.5/32       	trust
```
```bash
systemctl restart postgresql
```


### Configure Barman Server

```bash
vim /etc/barman.conf
```
```
[barman] 
barman_user = barman 
configuration_files_directory = /etc/barman.d 
barman_home = /var/lib/barman 
log_file = /var/log/barman/barman.log 
log_level = INFO 
compression = gzip
```
```bash
vim /etc/barman.d/pghost.conf
```
```
[pghost]
description = "Postgres Server Configuration"
ssh_command = ssh postgres@172.16.6.18
conninfo = host=172.16.6.18 user=barman dbname=postgres port=5432 password=barman
backup_method = rsync
reuse_backup = link
backup_options = concurrent_backup
parallel_jobs = 2
archiver = on
```

### How to Backup PostgreSQL Database From Barman Server

```
su barman

barman@bmhost:~$ barman list-server
pghost - Postgres Server Configuration

barman@bmhost:~$ barman check pghost
Server pghost:
    PostgreSQL: OK
    superuser or standard user with backup privileges: OK
    wal_level: OK
    directories: OK
    retention policy settings: OK
    backup maximum age: OK (no last_backup_maximum_age provided)
    backup minimum size: OK (33.6 MiB)
    wal maximum age: OK (no last_wal_maximum_age provided)
    wal size: OK (0 B)
    compression settings: OK
    failed backups: OK (there are 0 failed backups)
    minimum redundancy requirements: OK (have 6 backups, expected at least 0)
    ssh: OK (PostgreSQL server)
    systemid coherence: OK
    archive_mode: OK
    archive_command: OK
    continuous archiving: OK
    archiver errors: OK
```
```
barman@bmhost:~$ barman status pghost
Server pghost:
    Description: Postgres Server Configuration
    Active: True
    Disabled: False
    PostgreSQL version: 14.7
    Cluster state: in production
    Current data size: 34.0 MiB
    PostgreSQL Data directory: /var/lib/postgresql/14/main
    Current WAL segment: 000000010000000000000014
    PostgreSQL 'archive_command' setting: test ! -f barman@172.16.6.5:/var/lib/barman/pghost/incoming/%f && rsync -a %p barman@172.16.6.5:/var/lib/barman/pghost/incoming/%f
    Last archived WAL: 000000010000000000000013.00000028.backup, at Mon May  8 15:05:50 2023
    Failures of WAL archiver: 42 (000000010000000000000003 at Mon May  8 12:17:19 2023)
    Server WAL archiving rate: 0.47/hour
    Passive node: False
    Retention policies: not enforced
    No. of available backups: 6
    First available backup: 20230508T122427
    Last available backup: 20230508T150539
    Minimum redundancy requirements: satisfied (6/0)
```
```
barman@bmhost:~$ barman backup pghost
Starting backup using rsync-concurrent method for server pghost in /var/lib/barman/pghost/base/20230509T145211
Backup start at LSN: 0/15000028 (000000010000000000000015, 00000028)
Starting backup copy via rsync/SSH for 20230509T145211 (2 jobs)
Copy done (time: 4 seconds)
Asking PostgreSQL server to finalize the backup.
Backup size: 33.6 MiB. Actual size on disk: 147.9 KiB
Backup end at LSN: 0/15000138 (000000010000000000000015, 00000138)
Backup completed (start time: 2023-05-09 14:52:11.885550, elapsed time: 9 seconds)
Processing xlog segments from file archival for pghost
    000000010000000000000014
    000000010000000000000015
    000000010000000000000015.00000028.backup
```

```
barman@bmhost:~$ barman show-backup pghost 20230509T145211
Backup 20230509T145211:
  Server Name        	: pghost
  System Id          	: 7230380970858979530
  Status             	: DONE
  PostgreSQL Version 	: 140007
  PGDATA directory   	: /var/lib/postgresql/14/main

  Base backup information:
	Disk usage       	: 33.6 MiB (33.6 MiB with WALs)
	Incremental size 	: 147.9 KiB (-99.57%)
	Timeline         	: 1
	Begin WAL        	: 000000010000000000000015
	End WAL          	: 000000010000000000000015
	WAL number       	: 1
	WAL compression ratio: 99.90%
	Begin time       	: 2023-05-09 14:52:11.779790+06:00
	End time         	: 2023-05-09 14:52:18.681178+06:00
	Copy time        	: 4 seconds + 2 seconds startup
	Estimated throughput : 34.1 KiB/s (2 jobs)
	Begin Offset     	: 40
	End Offset       	: 312
	Begin LSN        	: 0/15000028
	End LSN          	: 0/15000138

  WAL information:
	No of files      	: 0
	Disk usage       	: 0 B
	Last available   	: 000000010000000000000015

  Catalog information:
	Retention Policy 	: not enforced
	Previous Backup  	: 20230508T150539
	Next Backup      	: - (this is the latest base backup)
```

#### Barman Backup Location

```
barman@bmhost:~$ ls -lah /var/lib/barman/pghost
```

#### How to Restore PostgreSQL Database From Barman Server

```
barman@bmhost:~$ barman show-backup pghost latest

Backup 20230509T145211:
  Server Name        	: pghost
  System Id          	: 7230380970858979530
  Status             	: DONE
  PostgreSQL Version 	: 140007
  PGDATA directory   	: /var/lib/postgresql/14/main

  Base backup information:
	Disk usage       	: 33.6 MiB (33.6 MiB with WALs)
	Incremental size 	: 147.9 KiB (-99.57%)
	Timeline         	: 1
	Begin WAL        	: 000000010000000000000015
	End WAL          	: 000000010000000000000015
	WAL number       	: 1
	WAL compression ratio: 99.90%
	Begin time       	: 2023-05-09 14:52:11.779790+06:00
	End time         	: 2023-05-09 14:52:18.681178+06:00
	Copy time        	: 4 seconds + 2 seconds startup
	Estimated throughput : 34.1 KiB/s (2 jobs)
	Begin Offset     	: 40
	End Offset       	: 312
	Begin LSN        	: 0/15000028
	End LSN          	: 0/15000138

  WAL information:
	No of files      	: 0
	Disk usage       	: 0 B
	Last available   	: 000000010000000000000015

  Catalog information:
	Retention Policy 	: not enforced
	Previous Backup  	: 20230508T150539
	Next Backup      	: - (this is the latest base backup)
```
```
barman@bmhost:~$  barman recover --target-time "2023-05-08 14:55:46.632938+06:00"  --remote-ssh-command "ssh postgres@172.16.6.26"   pghost   20230508T150539  /var/lib/postgresql/14/main
```

There are quite a few options, arguments, and variables here, so let’s explain them.
- `--target-time "Begin time"`: Use the begin time from the show-backup command
- `--remote-ssh-command "ssh postgres@standby-db-server-ip"`: Use the IP address of the standby-db-server
- `main-db-server`: Use the name of the database server from your /etc/barman.conf file
- `backup-id`: Use the backup ID from the show-backup command, or use latest if that’s the one you want
- `/var/lib/postgresql/14/main`: The path where you want the backup to be restored. This path will become the new data directory for Postgres on the standby server. Here, we have chosen the default data directory for Postgres in CentOS. For real-life use cases, choose the appropriate path






