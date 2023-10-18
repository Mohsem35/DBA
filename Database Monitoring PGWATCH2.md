Agendas

1. [Introduction](#introduction)
2. [Components](#components)
3. [Preparing databases for monitoring](#preparing-databases-for-monitoring)



### Introduction 

pgwatch2 is a flexible **`PostgreSQL-specific`** monitoring solution relying on **`Grafana`** dashboards for the UI part. It supports monitoring of almost all metrics for Postgres. At the core of the solution is the metrics gathering daemon written in **`Go`**

#### List of main features

- Non-invasive setup, no extensions nor superuser rights required for the base functionality
- Intuitive metrics presentation using the Grafana dashboarding engine with optional Alerting
- Lots of pre-configured dashboards and metric configurations covering all Statistics Collector data
- Easy extensibility by defining metrics in pure SQL (thus they could also be from business domain)
- _4 supported data stores for metrics storage (PostgreSQL with or without TimescaleDB, InfluxDB, Graphite, Prometheus)_
- Multiple configuration options (YAML, PostgreSQL, ENV) supporting both "push" and "pull" models
- _Possible to monitoring all or a subset of DBs of a PostgreSQL cluster_
- Global or DB level configuration of metrics/intervals
- Kubernetes/OpenShift ready with sample templates and a Helm chart
- PgBouncer, Pgpool-II, AWS RDS and Patroni support
- _Internal health-check API to monitor metrics gathering status_
- _Security options like SSL / HTTPS for all connections and password encryption for connect strings_
- _Very low resource requirements for the collector - 1 CPU core can handle ~3k monitored DBs at 1GB RAM usage_
- Log parsing capabilities when deployed locally in "push" mode

### Components

#### 1. The metrics gathering daemon

The metrics collector, written in Go, is the only **`mandatory and most critical component`** of the whole solution. The main task of the pgwatch2 collector / daemon is pretty simple - _1. reading the configuration and metric defintions_, _2.fetching the metrics from the configured databases_ using the configured connection info and _3. finally storing the metrics to some other database_, or just exposing them over a port for scraping in case of Prometheus mode.


#### 2. Configuration store

The configuration says **`which databases`**, **`how often`** and with **`which metrics (SQL-s queries)`** are to be gathered. There are 3 options to store the configuration:

1. _A PostgreSQL database_ holding a simple schema with 5 tables.
2. _File based approach_ - YAML config file(s) and SQL metric definition files.
3. _Purely ENV based setup_ - i.e. an “ad-hoc” config to monitor a single database or the whole instance. Bascially just a connect string (JDBC or Libpq type) is needed which is _perfect for “throwaway” and Cloud / container usage_



#### 3. Metrics storage DB

Many options here so that one can for example go for maximum storage effectiveness or pick something where they already know the query language:

1. _InfluxDB Time Series Database_
2. _PostgreSQL - world’s most advanced Open Source RDBMS_
3. _TimescaleDB time-series extension for PostgreSQL_
4. _Prometheus Time Series DB and monitoring system_
5. _Graphite Time Series DB_



#### 4. The Web UI

The second homebrewn component of the pgwatch2 solution is an optional and relatively simple Web UI for **`administering details`** of the monitoring configuration like _which databases should be monitored, with which metrics and intervals_. 


#### 5. Metrics representation

Standard pgwatch2 setup uses **`Grafana`** for analyzing the _gathered metrics data in a visual, point-and-click way_. For that a rich set of predefined dashboards for Postgres and InfluxDB data sources is provided, that should _cover the needs of most users_ - advanced users would mostly always want to customize some aspects though, so it’s not meant as a one-size-fits-all solution.

#### 6. Component reuse

NB! All components are **`loosely coupled`**, thus for non-pgwatch2 components (pgwatch2 components are only the metrics collector and the optional Web UI) you can decide to _make use of an already existing installation of Postgres, Grafana or InfluxDB and run additionally just the pgwatch2 collector_


### Installing using Docker

```
# let's create volumes for Postgres, Grafana and pgwatch2 marker files / SSL certificates
for v in pg  grafana pw2 ; do docker volume create $v ; done

# launch pgwatch2 with fully exposed Grafana and Health-check ports
# and local Postgres and subnet level Web UI ports
docker run -d --restart=unless-stopped --name pw2 \
  -p 3000:3000 -p 8081:8081 -p 127.0.0.1:5432:5432 -p 192.168.1.XYZ:8080:8080 \
  -v pg:/var/lib/postgresql -v grafana:/var/lib/grafana -v pw2:/pgwatch2/persistent-config \
  cybertec/pgwatch2-postgres:X.Y.Z
```

or Use `docker-compose` file

```
# docker-compose.yaml
version: '3'
services:
  pw2:
    container_name: pw2
    image: cybertec/pgwatch2-postgres:latest
    restart: always
    environment:
      - PW2_TESTDB=true
    ports:
      - "3000:3000"
      - "8081:8081"
      - "127.0.0.1:5432:5432"
      - "<host_machine_ip>:8080:8080"
    volumes:
      - pg:/var/lib/postgresql
      - grafana:/var/lib/grafana
      - pw2:/pgwatch2/persistent-config
    networks: 
      - bridge

volumes:
  pg:
  grafana:
  pw2:

networks:
  bridge:
```



5432 - Postgres configuration or metrics storage DB (when using the cybertec/pgwatch2-postgres image)

**`8080`** - _Management Web UI_ (monitored hosts, metrics, metrics configurations)

8081 - Gatherer healthcheck / statistics on number of gathered metrics (JSON).

**`3000`** - _Grafana dashboarding_

**`8086`** - _InfluxDB API_ (when using the InfluxDB version)

8088 - InfluxDB Backup port (when using the InfluxDB version)


##### Available Docker images

Following images are regularly pushed to Docked Hub:

**`cybertec/pgwatch2`**
    
The original pgwatch2 “batteries-included” image with __InfluxDB metrics storage__. Just insert connect infos to your database via the admin Web UI (or directly into the Config DB) and then turn to the pre-defined Grafana dashboards to analyze DB health and performance.

**`cybertec/pgwatch2-postgres`**

_Exactly the same as previous, but metrics are also stored in PostgreSQL_- thus needs more disk space. But in return you get more “out of the box” dashboards, as the power of standard SQL gives more complex visualization options.


### Preparing databases for monitoring

**Step 1**: Install pgwatch2 - either from pre-built packages or by compiling the Go code

**1st Approach**: Using pre-built packages

The pre-built DEB / RPM / Tar packages are available on the Github releases page.

```
# find out the latest package link and replace below, using v1.8.0 here
wget https://github.com/cybertec-postgresql/pgwatch2/releases/download/v1.8.0/pgwatch2_v1.8.0-SNAPSHOT-064fdaf_linux_64-bit.deb
sudo dpkg -i pgwatch2_v1.8.0-SNAPSHOT-064fdaf_linux_64-bit.deb
```    
Install Go by following the official instructions [Go Installation Offical Documentation](https://go.dev/doc/install)

**2nd Approach**: Compiling the **`Go`** code yourself

Download Go package

```
curl -OL https://golang.org/dl/go1.X.Y.linux-amd64.tar.gz
```

1. **Remove any previous Go installation** by deleting the /usr/local/go folder (if it exists), then extract the archive you just downloaded into /usr/local, creating a fresh Go tree in /usr/local/go: 
This method of course is not needed unless dealing with maximum security environments or some slight code changes are required.
```
sudo su
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.X.Y.linux-amd64.tar.gz
```
(You may need to run the command as root or through sudo). 
**Do not** untar the archive into an existing `/usr/local/go` tree. This is known to produce broken Go installations. 

2. Add `/usr/local/go/bin` to the **`PATH environment variable`**

You can do this by adding the following line to your $HOME/.profile or /etc/profile (for a system-wide installation):
```
export PATH=$PATH:/usr/local/go/bin
```

or Add permanently this variable 

```
vim .bashrc

# add the following line at the bottom of the .bashrc file
PATH=$PATH:/usr/local/go/bin

source .bashrc
```

3. **`Verify`** that you've installed Go by opening a command prompt and typing the following command:

```
go version
```
4. Confirm that the command prints the installed version of Go.
5. Get the **`pgwatch2 project’s code and compile`** the gatherer daemon

```bash
git clone https://github.com/cybertec-postgresql/pgwatch2.git
cd pgwatch2/pgwatch2
./build_gatherer.sh
# after fetching all the Go library dependencies (can take minutes)
# an executable named "pgwatch2" should be generated. Additionally it's a good idea
# to copy it to /usr/bin/pgwatch2-daemon as that's what the default SystemD service expects.
```

As a base requirement you'll need a login user (non-superuser suggested) for connecting to your PostgreSQL servers and fetching metrics queries.

```sql
CREATE ROLE pgwatch2 WITH LOGIN PASSWORD 'secret';
-- NB! For critical databases it might make sense to ensure that the user account
-- used for monitoring can only open a limited number of connections
-- (there are according checks in code, but multiple instances might be launched)
ALTER ROLE pgwatch2 CONNECTION LIMIT 3;
GRANT pg_monitor TO pgwatch2;   // v10+
GRANT CONNECT ON DATABASE mydb TO pgwatch2;
GRANT USAGE ON SCHEMA public TO pgwatch2; -- NB! pgwatch doesn't necessarily require using the public schema though!
GRANT EXECUTE ON FUNCTION pg_stat_file(text) to pgwatch2; -- needed by the wal_size metric
```



For most monitored databases it’s extremely beneficial (to troubleshooting performance issues) to also activate the `pg_stat_statements` extension which will give us exact `“per query”` performance aggregates and also enables to calculate how many queries are executed per second for example


1. Make sure the Postgres contrib package is installed (should be installed automatically together with the Postgres server package on Debian based systems).


- On RedHat / Centos: `yum install -y postgresqlXY-contrib`
- On Debian / Ubuntu: `apt install postgresql-contrib`


2. Add _pg_stat_statements_ to your server config (postgresql.conf) and restart the server.
```
shared_preload_libraries = 'pg_stat_statements'
track_io_timing = on
```

3. After restarting activate the extension in the monitored DB. Assumes Postgres superuser.

```
psql -c "CREATE EXTENSION IF NOT EXISTS pg_stat_statements"
```


#### Helper functions to retrieve protected statistics




Helper functions in pgwatch2 context are standard **`Postgres stored procedures`**, running under **`SECURITY DEFINER`** privileges. Via such wrapper functions one can do controlled privilege escalation - i.e. to give access to protected Postgres metrics (_like active session details, “per query” statistics_) or even OS-level metrics, to normal unprivileged users, like the pgwatch2 monitoring role.

##### Rolling out common helpers

```sql
export PGUSER=superuser
psql -f /etc/pgwatch2/metrics/00_helpers/get_stat_activity/$pgver/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_stat_replication/$pgver/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_wal_size/$pgver/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_stat_statements/$pgver/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_sequences/$pgver/metric.sql <db_name>
```

##### PL/Python helpers

```bash
# first install the Python bindings for Postgres
apt install postgresql-plpython3-XY
# yum install postgresqlXY-plpython3
```
```sql
psql -c "CREATE EXTENSION plpython3u"
psql -f /etc/pgwatch2/metrics/00_helpers/get_load_average/9.1/metric.sql <db_name>

# psutil helpers are only needed when full set of common OS metrics is wanted
apt install python3-psutil
psql -f /etc/pgwatch2/metrics/00_helpers/get_psutil_cpu/9.1/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_psutil_mem/9.1/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_psutil_disk/9.1/metric.sql <db_name>
psql -f /etc/pgwatch2/metrics/00_helpers/get_psutil_disk_io_total/9.1/metric.sql <db_name>
```




