#### 1st step : Stop replication among both primary and secondary server
 
1. Change the `postgresql.auto.conf` file both from master and standby server.
2. Remove the `standby.signal` file from standby server
3. Edit the `pg_hba.conf` file in primary server for removing the IP 
4. Edit `postgresql.conf` file to change the replication parameters 

Now, run the following command in the primary server

DROP A REPLICATION SLOT

```sql
select pg_drop_replication_slot(slot_name) from pg_replication_slots where slot_name = 'bottledwater';
```
OR
```sql
select pg_drop_replication_slot('slot_name');
```

#### 2nd step :  Create a ‘schema based‘ and ‘data-only’ dump from the database (Primary server) and save to any safe place, if any worst case occurs

```shell
ssh ubuntu@192.168.190.33

# for public schema-only
pg_dump --schema-only -U ibprod -d ibprod -n public > schemaonly_ibprod_public_$(date +%d-%m-%y).sql
# for public data-only
pg_dump -U ibprod -d ibprod --data-only -n public > dataonly_ibprod_public_$(date +%d-%m-%y).sql
# full public schema with data
pg_dump -U ibprod -d ibprod -n public > schema_ibprod_public_$(date +%d-%m-%y).sql

pg_dump -U ibprod -d ibprod -n mfs > schema_ibprod_mfs_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n nda > schema_ibprod_nda_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n academia > schema_ibprod_academia_$(date +%d-%m-%y).sql 
pg_dump -U ibprod -d ibprod -n top_up > schema_ibprod_topup_$(date +%d-%m-%y).sql   
pg_dump -U ibprod -d ibprod -n education > schema_ibprod_education_$(date +%d-%m-%y).sql
pg_dump -U ibprod -d ibprod -n bill > schema_ibprod_bill_$(date +%d-%m-%y).sql 
pg_dump -U ibprod -d ibprod -n bank_detail > schema_ibprod_bankdetail_$(date +%d-%m-%y).sql

# full dump
pg_dump -U ibprod -d ibprod > ibprod_$(date +%d-%m-%y).sql 
```


#### 3rd step : Create tablespace directory and change ownership (Primary server) 

In PostgreSQL, table partitioning does not strictly require the use of tablespaces. Tablespaces in PostgreSQL are used to organize database storage into different locations on disk, and they provide a way to control the physical storage of database objects.


```shell
sudo mkdir -p /var/data/postgres/ibprod/ibprodtbs
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2021


# Tablespace Directory Creation
# 2022
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m01
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m02
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m03
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m04
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m05
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m06
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m07
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m08
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m09
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m10
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m11
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2022m12

# 2023
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

# 2024
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

# 2025
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m01
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m02
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m03
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m04
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m05
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m06
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m07
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m08
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m09
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m10
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m11
sudo mkdir -p /var/data/postgres/ibprod/ibprod_logtbs_y2025m12


sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprodtbs
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2021


# Tablespace Directory Ownership Change 
# 2022
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m01
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m02
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m03
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m04
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m05
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m06
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m07
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m08
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m09
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m10
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m11
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2022m12

# 2023
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

# 2024
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

# 2025
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m01
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m02
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m03
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m04
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m05
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m06
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m07
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m08
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m09
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m10
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m11
sudo chown -R postgres:postgres /var/data/postgres/ibprod/ibprod_logtbs_y2025m12
```

> এতটুকু পর্যন্ত কাজ replica server তে ও করতে হবে না।

Give ‘postgres’ user sudo permission

```shell
usermod -a -G sudo postgres
```


#### 4th step : Create tablespace with postgres user(Primary Server)

```sql
DROP DATABASE ibprod;
CREATE DATABASE ibprod WITH OWNER = ibprod;
```
Execute the following queries as ‘Postgres’ superuser

```sql
drop tablespace if exists ibprodtbs; create tablespace ibprodtbs owner ibprod location '/var/data/postgres/ibprod/ibprodtbs';

-- 2021
drop tablespace if exists ibprod_logtbs_y2021; create tablespace ibprod_logtbs_y2021 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2021';

-- 2022
drop tablespace if exists ibprod_logtbs_y2022m01; create tablespace ibprod_logtbs_y2022m01 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m01';
drop tablespace if exists ibprod_logtbs_y2022m02; create tablespace ibprod_logtbs_y2022m02 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m02';
drop tablespace if exists ibprod_logtbs_y2022m03; create tablespace ibprod_logtbs_y2022m03 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m03';
drop tablespace if exists ibprod_logtbs_y2022m04; create tablespace ibprod_logtbs_y2022m04 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m04';
drop tablespace if exists ibprod_logtbs_y2022m05; create tablespace ibprod_logtbs_y2022m05 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m05';
drop tablespace if exists ibprod_logtbs_y2022m06; create tablespace ibprod_logtbs_y2022m06 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m06';
drop tablespace if exists ibprod_logtbs_y2022m07; create tablespace ibprod_logtbs_y2022m07 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m07';
drop tablespace if exists ibprod_logtbs_y2022m08; create tablespace ibprod_logtbs_y2022m08 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m08';
drop tablespace if exists ibprod_logtbs_y2022m09; create tablespace ibprod_logtbs_y2022m09 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m09';
drop tablespace if exists ibprod_logtbs_y2022m10; create tablespace ibprod_logtbs_y2022m10 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m10';
drop tablespace if exists ibprod_logtbs_y2022m11; create tablespace ibprod_logtbs_y2022m11 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m11';
drop tablespace if exists ibprod_logtbs_y2022m12; create tablespace ibprod_logtbs_y2022m12 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2022m12';

-- 2023
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

-- 2024
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

-- 2025 
drop tablespace if exists ibprod_logtbs_y2025m01; create tablespace ibprod_logtbs_y2025m01 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m01';
drop tablespace if exists ibprod_logtbs_y2025m02; create tablespace ibprod_logtbs_y2025m02 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m02';
drop tablespace if exists ibprod_logtbs_y2025m03; create tablespace ibprod_logtbs_y2025m03 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m03';
drop tablespace if exists ibprod_logtbs_y2025m04; create tablespace ibprod_logtbs_y2025m04 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m04';
drop tablespace if exists ibprod_logtbs_y2025m05; create tablespace ibprod_logtbs_y2025m05 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m05';
drop tablespace if exists ibprod_logtbs_y2025m06; create tablespace ibprod_logtbs_y2025m06 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m06';
drop tablespace if exists ibprod_logtbs_y2025m07; create tablespace ibprod_logtbs_y2025m07 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m07';
drop tablespace if exists ibprod_logtbs_y2025m08; create tablespace ibprod_logtbs_y2025m08 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m08';
drop tablespace if exists ibprod_logtbs_y2025m09; create tablespace ibprod_logtbs_y2025m09 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m09';
drop tablespace if exists ibprod_logtbs_y2025m10; create tablespace ibprod_logtbs_y2025m10 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m10';
drop tablespace if exists ibprod_logtbs_y2025m11; create tablespace ibprod_logtbs_y2025m11 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m11';
drop tablespace if exists ibprod_logtbs_y2025m12; create tablespace ibprod_logtbs_y2025m12 owner ibprod location '/var/data/postgres/ibprod/ibprod_logtbs_y2025m12';
```


#### 5th Step: Now edit the public.transaction table in the --schema-only dump file(sql dump) and restore

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
   beftn_reconcile_oid varchar(255) NULL,
   clienttype varchar(32) NULL DEFAULT 'app'::character varying
   -- CONSTRAINT ck_transstatus_transaction CHECK ((((transstatus)::text = 'RequestReceived'::text) OR ((transstatus)::text = 'Pending'::text) OR ((transstatus)::text = 'Failed'::text) OR ((transstatus)::text = 'OK'::text) OR ((transstatus)::text = 'Reversed'::text))),
   -- CONSTRAINT pk_transaction PRIMARY KEY (transactionoid)
) PARTITION BY RANGE (createdon);
```

#### 6th Step: Restore every schema to the newly created database (Primary) 

```shell
# in public schema no data is restored, just restore schema-only structure
psql -U ibprod -d ibprod -n public < schemaonly_ibprod_public_$(date +%d-%m-%y).sql

# restoration with data
psql -U ibprod -d ibprod -n mfs < schema_ibprod_mfs_$(date +%d-%m-%y).sql
psql -U ibprod -d ibprod -n nda < schema_ibprod_nda_$(date +%d-%m-%y).sql
psql -U ibprod -d ibprod -n academia < schema_ibprod_academia_$(date +%d-%m-%y).sql 
psql -U ibprod -d ibprod -n top_up < schema_ibprod_topup_$(date +%d-%m-%y).sql   
psql -U ibprod -d ibprod -n education < schema_ibprod_education_$(date +%d-%m-%y).sql
psql -U ibprod -d ibprod -n bill < schema_ibprod_bill_$(date +%d-%m-%y).sql 
psql -U ibprod -d ibprod -n bank_detail < schema_ibprod_bankdetail_$(date +%d-%m-%y).sql
```


#### 7th step : Create table partition for public.transaction

```shell
psql -U ibprod -d ibprod
```

```sql
create table transaction_y2021 partition of transaction for values from ('2021-01-01 00:00:00') TO ('2021-12-31 23:59:59') tablespace ibprod_logtbs_y2021;


-- 2022
create table transaction_y2022m01 partition of transaction for values from ('2022-01-01 00:00:00') TO ('2022-01-31 23:59:59') tablespace ibprod_logtbs_y2022m01;
create table transaction_y2022m02 partition of transaction for values from ('2022-02-01 00:00:00') TO ('2022-02-28 23:59:59') tablespace ibprod_logtbs_y2022m02;
create table transaction_y2022m03 partition of transaction for values from ('2022-03-01 00:00:00') TO ('2022-03-31 23:59:59')tablespace ibprod_logtbs_y2022m03;
create table transaction_y2022m04 partition of transaction for values from ('2022-04-01 00:00:00') TO ('2022-04-30 23:59:59')tablespace ibprod_logtbs_y2022m04;
create table transaction_y2022m05 partition of transaction for values from ('2022-05-01 00:00:00') TO ('2022-05-31 23:59:59') tablespace ibprod_logtbs_y2022m05;
create table transaction_y2022m06 partition of transaction for values from ('2022-06-01 00:00:00') TO ('2022-06-30 23:59:59') tablespace ibprod_logtbs_y2022m06;
create table transaction_y2022m07 partition of transaction for values from ('2022-07-01 00:00:00') TO ('2022-07-31 23:59:59') tablespace ibprod_logtbs_y2022m07;
create table transaction_y2022m08 partition of transaction for values from ('2022-08-01 00:00:00') TO ('2022-08-31 23:59:59') tablespace ibprod_logtbs_y2022m08;
create table transaction_y2022m09 partition of transaction for values from ('2022-09-01 00:00:00') TO ('2022-09-30 23:59:59') tablespace ibprod_logtbs_y2022m09;
create table transaction_y2022m10 partition of transaction for values from ('2022-10-01 00:00:00') TO ('2022-10-31 23:59:59') tablespace ibprod_logtbs_y2022m10;
create table transaction_y2022m11 partition of transaction for values from ('2022-11-01 00:00:00') TO ('2022-11-30 23:59:59') tablespace ibprod_logtbs_y2022m11;
create table transaction_y2022m12 partition of transaction for values from ('2022-12-01 00:00:00') TO ('2022-12-31 23:59:59') tablespace ibprod_logtbs_y2022m12;

-- 2023
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

-- 2024 (leap year, February 29 day)
create table transaction_y2024m01 partition of transaction for values from ('2024-01-01 00:00:00') TO ('2024-01-31 23:59:59') tablespace ibprod_logtbs_y2024m01;
create table transaction_y2024m02 partition of transaction for values from ('2024-02-01 00:00:00') TO ('2024-02-29 23:59:59') tablespace ibprod_logtbs_y2024m02;
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

-- 2025
create table transaction_y2025m01 partition of transaction for values from ('2025-01-01 00:00:00') TO ('2025-01-31 23:59:59') tablespace ibprod_logtbs_y2025m01;
create table transaction_y2025m02 partition of transaction for values from ('2025-02-01 00:00:00') TO ('2025-02-28 23:59:59') tablespace ibprod_logtbs_y2025m02;
create table transaction_y2025m03 partition of transaction for values from ('2025-03-01 00:00:00') TO ('2025-03-31 23:59:59')tablespace ibprod_logtbs_y2025m03;
create table transaction_y2025m04 partition of transaction for values from ('2025-04-01 00:00:00') TO ('2025-04-30 23:59:59')tablespace ibprod_logtbs_y2025m04;
create table transaction_y2025m05 partition of transaction for values from ('2025-05-01 00:00:00') TO ('2025-05-31 23:59:59') tablespace ibprod_logtbs_y2025m05;
create table transaction_y2025m06 partition of transaction for values from ('2025-06-01 00:00:00') TO ('2025-06-30 23:59:59') tablespace ibprod_logtbs_y2025m06;
create table transaction_y2025m07 partition of transaction for values from ('2025-07-01 00:00:00') TO ('2025-07-31 23:59:59') tablespace ibprod_logtbs_y2025m07;
create table transaction_y2025m08 partition of transaction for values from ('2025-08-01 00:00:00') TO ('2025-08-31 23:59:59') tablespace ibprod_logtbs_y2025m08;
create table transaction_y2025m09 partition of transaction for values from ('2025-09-01 00:00:00') TO ('2025-09-30 23:59:59') tablespace ibprod_logtbs_y2025m09;
create table transaction_y2025m10 partition of transaction for values from ('2025-10-01 00:00:00') TO ('2025-10-31 23:59:59') tablespace ibprod_logtbs_y2025m10;
create table transaction_y2025m11 partition of transaction for values from ('2025-11-01 00:00:00') TO ('2025-11-30 23:59:59') tablespace ibprod_logtbs_y2025m11;
create table transaction_y2025m12 partition of transaction for values from ('2025-12-01 00:00:00') TO ('2025-12-31 23:59:59') tablespace ibprod_logtbs_y2025m12;

```

#### 8th step : Restore that data-only dump of public.transaction table

```shell
psql -U ibprod -d ibprod -n public < dataonly_ibprod_public_$(date +%d-%m-%y).sql
```
```shell
# Another Approach (Tanim vai procedure)
pg_dump --username=ibprod --dbname=ibprod --data-only --jobs=4 --encoding=UTF8 --format=d --file=/home/ubuntu/dump
pg_restore --username=ibprod --dbname=ibprod --clean --if-exists --exit-on-error --verbose /home/ubuntu/dump
```

#### 9th Step: Indexing the table

```sql
create index transaction_y2021_transactioncreatedon on transaction_y2021 (createdon);

-- 2022
create index transaction_y2022m01_transactioncreatedon on transaction_y2022m01 (createdon);
create index transaction_y2022m02_transactioncreatedon on transaction_y2022m02 (createdon);
create index transaction_y2022m03_transactioncreatedon on transaction_y2022m03 (createdon);
create index transaction_y2022m04_transactioncreatedon on transaction_y2022m04 (createdon);
create index transaction_y2022m05_transactioncreatedon on transaction_y2022m05 (createdon);
create index transaction_y2022m06_transactioncreatedon on transaction_y2022m06 (createdon);
create index transaction_y2022m07_transactioncreatedon on transaction_y2022m07 (createdon);
create index transaction_y2022m08_transactioncreatedon on transaction_y2022m08 (createdon);
create index transaction_y2022m09_transactioncreatedon on transaction_y2022m09 (createdon);
create index transaction_y2022m10_transactioncreatedon on transaction_y2022m10 (createdon);
create index transaction_y2022m11_transactioncreatedon on transaction_y2022m11 (createdon);
create index transaction_y2022m12_transactioncreatedon on transaction_y2022m12 (createdon);


-- 2023
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

-- 2024
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

-- 2025
create index transaction_y2025m01_transactioncreatedon on transaction_y2025m01 (createdon);
create index transaction_y2025m02_transactioncreatedon on transaction_y2025m02 (createdon);
create index transaction_y2025m03_transactioncreatedon on transaction_y2025m03 (createdon);
create index transaction_y2025m04_transactioncreatedon on transaction_y2025m04 (createdon);
create index transaction_y2025m05_transactioncreatedon on transaction_y2025m05 (createdon);
create index transaction_y2025m06_transactioncreatedon on transaction_y2025m06 (createdon);
create index transaction_y2025m07_transactioncreatedon on transaction_y2025m07 (createdon);
create index transaction_y2025m08_transactioncreatedon on transaction_y2025m08 (createdon);
create index transaction_y2025m09_transactioncreatedon on transaction_y2025m09 (createdon);
create index transaction_y2025m10_transactioncreatedon on transaction_y2025m10 (createdon);
create index transaction_y2025m11_transactioncreatedon on transaction_y2025m11 (createdon);
create index transaction_y2025m12_transactioncreatedon on transaction_y2025m12 (createdon);
```

```sql
check indexing
# psql
\d transaction_y2022
```

After completing all the procedure, I will set up the replication again
