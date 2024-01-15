In PostgreSQL, table partitioning does not strictly require the use of tablespaces. Tablespaces in PostgreSQL are used to organize database storage into different locations on disk, and they provide a way to control the physical storage of database objects.


#### 1st step : Create tablespace directory and change ownership

```shell
sudo mkdir -p /var/data/postgres/ibprod/ibprodtbs
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2021
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m01
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m02
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m03
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m04
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m05
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m06
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m07
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m08
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m09
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m10
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m11
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2023m12
```

```shell
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprodtbs
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2021
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m01
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m02
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m03
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m04
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m05
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m06
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m07
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m08
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m09
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m10
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m11
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2023m12
```
> এতটুকু পর্যন্ত কাজ replica server তে ও করতে হবে না। 

```shell
usermod -a -G sudo postgres
```

#### 2nd step : create tablespace with postgres user

```shell
DROP DATABASE ibprod;
CREATE DATABASE ibprod WITH OWNER = ibprod;
```

Execute the following queries as Postgres superuser

```sql
drop tablespace if exists ibprod_logtbs_y2023m01; create tablespace ibprod_logtbs_y2023m01 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m01';
drop tablespace if exists ibprod_logtbs_y2023m02; create tablespace ibprod_logtbs_y2023m02 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m02';
drop tablespace if exists ibprod_logtbs_y2023m03; create tablespace ibprod_logtbs_y2023m03 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m03';
drop tablespace if exists ibprod_logtbs_y2023m04; create tablespace ibprod_logtbs_y2023m04 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m04';
drop tablespace if exists ibprod_logtbs_y2023m05; create tablespace ibprod_logtbs_y2023m05 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m05';
drop tablespace if exists ibprod_logtbs_y2023m06; create tablespace ibprod_logtbs_y2023m06 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m06';
drop tablespace if exists ibprod_logtbs_y2023m07; create tablespace ibprod_logtbs_y2023m07 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m07';
drop tablespace if exists ibprod_logtbs_y2023m08; create tablespace ibprod_logtbs_y2023m08 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m08';
drop tablespace if exists ibprod_logtbs_y2023m09; create tablespace ibprod_logtbs_y2023m09 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m09';
drop tablespace if exists ibprod_logtbs_y2023m10; create tablespace ibprod_logtbs_y2023m10 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m10';
drop tablespace if exists ibprod_logtbs_y2023m11; create tablespace ibprod_logtbs_y2023m11 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m11';
drop tablespace if exists ibprod_logtbs_y2023m12; create tablespace ibprod_logtbs_y2023m12 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2023m12';
drop tablespace if exists ibprodtbs; create tablespace ibprodtbs owner ibprod location '/var/data/postgres/ibprod/ibprodtbs';
drop tablespace if exists ibprod_logtbs_y2021; create tablespace ibprod_logtbs_y2021 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2021';
drop tablespace if exists ibprod_logtbs_y2022; create tablespace ibprod_logtbs_y2022 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022';
```

#### 3rd Step: Create a schema-only dump from the database 


```shell
# dump without data
pg_dump --schema-only -U ibprod -d ibprod -n public > schema_ibprod_public_$(date +%d-%m-%y).sql

# dump with data
pg_dump -U ibprod -d ibprod -n nda > schema_nda_ibprod_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n mfs > schema_mfs_ibprod_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n bill > schema_bill_ibprod_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n bank_detail > schema_bankdetail_ibprod_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n academia > schema_academia_ibprod_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n education > schema_mfs_ibprod_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n top_up > schema_topup_ibprod_$(date +%d-%m-%y).sql
```

#### 4th Step: Now modify the `transaction` table in the dump file and restore

```sql

-- public."transaction" definition

-- Drop table

-- DROP TABLE public."transaction";

CREATE TABLE public."transaction" (
	transactionoid varchar(128) NOT NULL,
	transtype varchar(32) NOT NULL,
	transamount numeric(20, 6) NOT NULL,
	chargeamount numeric(20, 6) NULL,
	vatamount numeric(20, 6) NULL,
	transstatus varchar(32) NOT NULL,
	failurereason text NULL,
	bankoid varchar(128) NULL,
	requestid varchar(128) NULL,
	cbsreferenceno varchar(64) NULL,
	narration varchar(256) NULL,
	remarks varchar(256) NULL,
	traceid varchar(128) NOT NULL,
	debitedaccountoid varchar(128) NULL,
	debitedaccount varchar(64) NULL,
	debitedaccountcustomeroid varchar(128) NULL,
	debitedaccountcbscustomerid varchar(128) NULL,
	debitedaccountbranchoid varchar(128) NULL,
	creditedaccountoid varchar(128) NULL,
	creditedaccount varchar(64) NULL,
	creditedaccountcustomeroid varchar(128) NULL,
	creditedaccountcbscustomerid varchar(128) NULL,
	creditedaccountbranchoid varchar(128) NULL,
	createdby varchar(128) NOT NULL DEFAULT 'System'::character varying,
	createdon timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
	editedby varchar(128) NULL,
	editedon timestamp NULL,
	offsetotp varchar(16) NULL,
	currency varchar(16) NULL,
	otpresentcount varchar NULL DEFAULT 0,
	cbstraceno varchar(128) NULL,
	cbstransactiondate timestamp NULL,
	transactionrequestdate timestamp NULL,
	otpsenton timestamp NULL,
	otpverifiedon timestamp NULL,
	ibuseroid varchar(128) NULL,
	debitedaccounttitle varchar(256) NULL,
	creditedaccounttitle varchar(256) NULL,
	intermediatestatus text NULL,
	customeraccount varchar(64) NULL,
	servicename varchar(256) NULL,
	customerbankoid varchar(128) NULL,
	beftn_reconcile_status varchar NULL,
	beftn_reconcile_oid varchar(255) NULL
	--CONSTRAINT ck_transstatus_transaction CHECK ((((transstatus)::text = 'RequestReceived'::text) OR ((transstatus)::text = 'Pending'::text) OR ((transstatus)::text = 'Failed'::text) OR ((transstatus)::text = 'OK'::text) OR ((transstatus)::text = 'Reversed'::text))),
	--CONSTRAINT pk_transaction PRIMARY KEY (transactionoid)
) PARTITION BY RANGE (createdon);
```

> যেই table modify করতে হবে, ওই table যেই schema তে থাকবে শুধুমাত্র সেই schema টা আমরা `schema-only` dump নিব. এইদিকে যেমন transaction table modify করতে হবে। তাহলে আমাদের public schema schema-only dump নিব  

```shell
psql -U ibprod -d ibprod < schema_ibprod_public_$(date +%d-%m-%y).sql


psql -U ibprod -d ibprod -n nda < schema_nda_ibprod_$(date +%d-%m-%y).sql
psql -U ibprod -d ibprod -n mfs < schema_mfs_ibprod_$(date +%d-%m-%y).sql
...
...
# do the same thing for other schemas
```



#### 5th step : Create table partition

```sql
create table transaction_y2021 partition of transaction for values from ('2021-01-01 00:00:00') TO ('2021-12-31 23:59:59') tablespace ibprod_logtbs_y2021;
create table transaction_y2022 partition of transaction for values from ('2022-01-01 00:00:00') TO ('2022-12-31 23:59:59') tablespace ibprod_logtbs_y2022;
create table transaction_y2023m01 partition of transaction for values from ('2023-01-01 00:00:00') TO ('2023-01-31 23:59:59') tablespace ibprod_logtbs_y2023m01;
create table transaction_y2023m02 partition of transaction for values from ('2023-02-01 00:00:00') TO ('2023-02-28 23:59:59') tablespace ibprod_logtbs_y2023m02;
create table transaction_y2023m03 partition of transaction for values from ('2023-03-01 00:00:00') TO ('2023-03-31 23:59:59')tablespace ibprod_logtbs_y2023m03;
create table transaction_y2023m04 partition of transaction for values from ('2023-04-01 00:00:00') TO ('2023-04-30 23:59:59')tablespace ibprod_logtbs_y2023m04;
create table transaction_y2023m05 partition of transaction for values from ('2023-05-01 00:00:00') TO ('2023-05-31 23:59:59') tablespace ibprod_logtbs_y2023m05;
create table transaction_y2023m06 partition of transaction for values from ('2023-06-01 00:00:00') TO ('2023-06-30 23:59:59') tablespace ibprod_logtbs_y2023m06;
create table transaction_y2023m07 partition of transaction for values from ('2023-07-01 00:00:00') TO ('2023-07-31 23:59:59') tablespace ibprod_logtbs_y2023m07;
create table transaction_y2023m08 partition of transaction for values from ('2023-08-01 00:00:00') TO ('2023-08-31 23:59:59') tablespace ibprod_logtbs_y2023m08;
create table transaction_y2023m09 partition of transaction for values from ('2023-09-01 00:00:00') TO ('2023-09-30 23:59:59') tablespace ibprod_logtbs_y2023m09;
create table transaction_y2023m10 partition of transaction for values from ('2023-10-01 00:00:00') TO ('2023-10-31 23:59:59') tablespace ibprod_logtbs_y2023m10;
create table transaction_y2023m11 partition of transaction for values from ('2023-11-01 00:00:00') TO ('2023-11-30 23:59:59') tablespace ibprod_logtbs_y2023m11;
create table transaction_y2023m12 partition of transaction for values from ('2023-12-01 00:00:00') TO ('2023-12-31 23:59:59') tablespace ibprod_logtbs_y2023m12;
```





#### 6th step : Create a data-only dump and restore that dump 

```shell
pg_dump -U ibprod -d ibprod --data-only -n public > ibprod_dataonly.sql
```
```shell
psql -U ibprod -d ibprod < ibprod_dataonly.sq
```


```shell
# Tanim vai procedure
pg_dump --username=ibprod --dbname=ibprod --data-only --jobs=4 --encoding=UTF8 --format=d --file=/home/ubuntu/dump
pg_restore --username=ibprod --dbname=ibprod --clean --if-exists --exit-on-error --verbose /home/ubuntu/dump
```

#### 7th Step: Indexing

```sql
create index transaction_y2021_transactioncreatedon on transaction_y2021 (createdon);
create index transaction_y2022_transactioncreatedon on transaction_y2022 (createdon);
create index transaction_y2023m01_transactioncreatedon on transaction_y2023m01 (createdon);
create index transaction_y2023m02_transactioncreatedon on transaction_y2023m02 (createdon);
create index transaction_y2023m03_transactioncreatedon on transaction_y2023m03 (createdon);
create index transaction_y2023m04_transactioncreatedon on transaction_y2023m04 (createdon);
create index transaction_y2023m05_transactioncreatedon on transaction_y2023m05 (createdon);
create index transaction_y2023m06_transactioncreatedon on transaction_y2023m06 (createdon);
create index transaction_y2023m07_transactioncreatedon on transaction_y2023m07 (createdon);
create index transaction_y2023m08_transactioncreatedon on transaction_y2023m08 (createdon);
create index transaction_y2023m09_transactioncreatedon on transaction_y2023m09 (createdon);
create index transaction_y2023m10_transactioncreatedon on transaction_y2023m10 (createdon);
create index transaction_y2023m11_transactioncreatedon on transaction_y2023m11 (createdon);
create index transaction_y2023m12_transactioncreatedon on transaction_y2023m12 (createdon);
```

**check indexing**

```sql
# psql
\d transaction_y2022
```
I am executing the pg_basebackup command for replication as a root user. so all the files and tablespace directories and postgres data directory will be copied from master as root. so need to change the ownership after coping all files.



### Issues 

replication করার পরে must ownership change করতে হবে


```shell
chown -R postgres:postgres /var/data/*
chown -R postgres:postgres /var/lib/postgresql/14/main/*
```

### New Requirments

We need to make new tablespaces for year 2024(monthly)

We will do the following tasks for both primary and secondary server(if configured)

```shell
sudo su
```

```shell
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m01
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m02
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m03
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m04
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m05
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m06
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m07
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m08
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m09
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m10
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m11
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2024m12
```

```shell
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m01
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m02
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m03
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m04
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m05
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m06
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m07
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m08
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m09
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m10
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m11
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2024m12
```
```shell
sudo su - postgres
psql
```

```sql
drop tablespace if exists ibprod_logtbs_y2024m01; create tablespace ibprod_logtbs_y2024m01 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m01';
drop tablespace if exists ibprod_logtbs_y2024m02; create tablespace ibprod_logtbs_y2024m02 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m02';
drop tablespace if exists ibprod_logtbs_y2024m03; create tablespace ibprod_logtbs_y2024m03 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m03';
drop tablespace if exists ibprod_logtbs_y2024m04; create tablespace ibprod_logtbs_y2024m04 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m04';
drop tablespace if exists ibprod_logtbs_y2024m05; create tablespace ibprod_logtbs_y2024m05 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m05';
drop tablespace if exists ibprod_logtbs_y2024m06; create tablespace ibprod_logtbs_y2024m06 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m06';
drop tablespace if exists ibprod_logtbs_y2024m07; create tablespace ibprod_logtbs_y2024m07 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m07';
drop tablespace if exists ibprod_logtbs_y2024m08; create tablespace ibprod_logtbs_y2024m08 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m08';
drop tablespace if exists ibprod_logtbs_y2024m09; create tablespace ibprod_logtbs_y2024m09 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m09';
drop tablespace if exists ibprod_logtbs_y2024m10; create tablespace ibprod_logtbs_y2024m10 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m10';
drop tablespace if exists ibprod_logtbs_y2024m11; create tablespace ibprod_logtbs_y2024m11 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m11';
drop tablespace if exists ibprod_logtbs_y2024m12; create tablespace ibprod_logtbs_y2024m12 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2024m12';


create table transaction_y2024m01 partition of transaction for values from ('2024-01-01 00:00:00') TO ('2024-01-31 23:59:59') tablespace ibprod_logtbs_y2024m01;
create table transaction_y2024m02 partition of transaction for values from ('2024-02-01 00:00:00') TO ('2024-02-28 23:59:59') tablespace ibprod_logtbs_y2024m02;
create table transaction_y2024m03 partition of transaction for values from ('2024-03-01 00:00:00') TO ('2024-03-31 23:59:59')tablespace ibprod_logtbs_y2024m03;
create table transaction_y2024m04 partition of transaction for values from ('2024-04-01 00:00:00') TO ('2024-04-30 23:59:59')tablespace ibprod_logtbs_y2024m04;
create table transaction_y2024m05 partition of transaction for values from ('2024-05-01 00:00:00') TO ('2024-05-31 23:59:59') tablespace ibprod_logtbs_y2024m05;
create table transaction_y2024m06 partition of transaction for values from ('2024-06-01 00:00:00') TO ('2024-06-30 23:59:59') tablespace ibprod_logtbs_y2024m06;
create table transaction_y2024m07 partition of transaction for values from ('2024-07-01 00:00:00') TO ('2024-07-31 23:59:59') tablespace ibprod_logtbs_y2024m07;
create table transaction_y2024m08 partition of transaction for values from ('2024-08-01 00:00:00') TO ('2024-08-31 23:59:59') tablespace ibprod_logtbs_y2024m08;
create table transaction_y2024m09 partition of transaction for values from ('2024-09-01 00:00:00') TO ('2024-09-30 23:59:59') tablespace ibprod_logtbs_y2024m09;
create table transaction_y2024m10 partition of transaction for values from ('2024-10-01 00:00:00') TO ('2024-10-31 23:59:59') tablespace ibprod_logtbs_y2024m10;
create table transaction_y2024m11 partition of transaction for values from ('2024-11-01 00:00:00') TO ('2024-11-30 23:59:59') tablespace ibprod_logtbs_y2024m11;
create table transaction_y2024m12 partition of transaction for values from ('2024-12-01 00:00:00') TO ('2024-12-31 23:59:59') tablespace ibprod_logtbs_y2024m12;


create index transaction_y2024m01_transactioncreatedon on transaction_y2024m01 (createdon);
create index transaction_y2024m02_transactioncreatedon on transaction_y2024m02 (createdon);
create index transaction_y2024m03_transactioncreatedon on transaction_y2024m03 (createdon);
create index transaction_y2024m04_transactioncreatedon on transaction_y2024m04 (createdon);
create index transaction_y2024m05_transactioncreatedon on transaction_y2024m05 (createdon);
create index transaction_y2024m06_transactioncreatedon on transaction_y2024m06 (createdon);
create index transaction_y2024m07_transactioncreatedon on transaction_y2024m07 (createdon);
create index transaction_y2024m08_transactioncreatedon on transaction_y2024m08 (createdon);
create index transaction_y2024m09_transactioncreatedon on transaction_y2024m09 (createdon);
create index transaction_y2024m10_transactioncreatedon on transaction_y2024m10 (createdon);
create index transaction_y2024m11_transactioncreatedon on transaction_y2024m11 (createdon);
create index transaction_y2024m12_transactioncreatedon on transaction_y2024m12 (createdon);
```
> _Note_: After doing the same tasks in secondary server, we may need to start/restart the secondary server

```sql
pg_dump --schema-only -h 127.0.0.1 -U mraims -d mraims -n template > schema_mraims_template_07_oct_4_00_pm.sql
```

