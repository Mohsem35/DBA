### Introduction 

pgwatch2 is a flexible **`PostgreSQL-specific`** monitoring solution relying on **`Grafana`** dashboards for the UI part. It supports monitoring of almost all metrics for Postgres. At the core of the solution is the metrics gathering daemon written in **`Go`**

### List of main features

- Non-invasive setup on PostgreSQL side - **`no extensions nor superuser rights are required`** for the base functionality so that even unprivileged users like developers can get a good overview of database activities without any hassle
- **`Many metric data storage options`** - _PostgreSQL_, _PostgreSQL with the compression enabled TimescaleDB extension_, _InfluxDB_, _Graphite_ or Prometheus scraping
- Possible to monitoring all, single or a subset (list or regex) of databases of a PostgreSQL instance
- Capabilities to go beyond PostgreSQL metrics gathering with built-in log parsing for error detection and **`OS level`** _metrics collection_ via **`PL/Python`** “helper” stored procedures
- A _Ping_ mode to test connectivity to all databases under monitoring


### Components

#### The metrics gathering daemon

The metrics collector, written in Go, is the only **`mandatory and most critical component`** of the whole solution. The main task of the pgwatch2 collector / daemon is pretty simple - _1. reading the configuration and metric defintions_, _2.fetching the metrics from the configured databases_ using the configured connection info and _3. finally storing the metrics to some other database_, or just exposing them over a port for scraping in case of Prometheus mode.



#### Configuration store

The configuration says **`which databases`**, **`how often`** and with **`which metrics (SQL-s queries)`** are to be gathered. There are 3 options to store the configuration:

1. _A PostgreSQL database_ holding a simple schema with 5 tables.
2. _File based approach_ - YAML config file(s) and SQL metric definition files.
3. _Purely ENV based setup_ - i.e. an “ad-hoc” config to monitor a single database or the whole instance. Bascially just a connect string (JDBC or Libpq type) is needed which is _perfect for “throwaway” and Cloud / container usage_



#### Metrics storage DB

Many options here so that one can for example go for maximum storage effectiveness or pick something where they already know the query language:

1. _InfluxDB Time Series Database_
2. _PostgreSQL - world’s most advanced Open Source RDBMS_
3. _TimescaleDB time-series extension for PostgreSQL_
4. _Prometheus Time Series DB and monitoring system_
5. _Graphite Time Series DB_



#### The Web UI

The second homebrewn component of the pgwatch2 solution is an optional and relatively simple Web UI for **`administering details`** of the monitoring configuration like _which databases should be monitored, with which metrics and intervals_. 


#### Metrics representation

Standard pgwatch2 setup uses **`Grafana`** for analyzing the _gathered metrics data in a visual, point-and-click way_. For that a rich set of predefined dashboards for Postgres and InfluxDB data sources is provided, that should _cover the needs of most users_ - advanced users would mostly always want to customize some aspects though, so it’s not meant as a one-size-fits-all solution.

#### Component reuse

NB! All components are **`loosely coupled`**, thus for non-pgwatch2 components (pgwatch2 components are only the metrics collector and the optional Web UI) you can decide to _make use of an already existing installation of Postgres, Grafana or InfluxDB and run additionally just the pgwatch2 collector_



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
**`8086`** - InfluxDB API (when using the InfluxDB version)
8088 - InfluxDB Backup port (when using the InfluxDB version)


##### Available Docker images

Following images are regularly pushed to Docked Hub:

**`cybertec/pgwatch2`**
    
The original pgwatch2 “batteries-included” image with __InfluxDB metrics storage__. Just insert connect infos to your database via the admin Web UI (or directly into the Config DB) and then turn to the pre-defined Grafana dashboards to analyze DB health and performance.

**`cybertec/pgwatch2-postgres`**

_Exactly the same as previous, but metrics are also stored in PostgreSQL_- thus needs more disk space. But in return you get more “out of the box” dashboards, as the power of standard SQL gives more complex visualization options.




