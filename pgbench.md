# pgbench: Benchmarking PostgreSQL

A practical guide to `pgbench`, PostgreSQL's built-in benchmarking tool: installing it, building a test database, running benchmarks, reading the results, writing realistic custom workloads, and measuring what pgbench itself does not report.

> **Version:** written for **PostgreSQL 18** (current release, 18.x). Everything here also works on PostgreSQL 15–17 unless noted. PostgreSQL 19 is in beta at the time of writing.

---

## Table of contents

1. [What pgbench is (and is not)](#1-what-pgbench-is-and-is-not)
2. [Install pgbench](#2-install-pgbench)
3. [Create an isolated lab cluster](#3-create-an-isolated-lab-cluster)
4. [Initialize the test database](#4-initialize-the-test-database)
5. [Run a benchmark](#5-run-a-benchmark)
6. [Read the output](#6-read-the-output)
7. [Built-in scripts](#7-built-in-scripts)
8. [Custom scripts for realistic workloads](#8-custom-scripts-for-realistic-workloads)
9. [Latency percentiles from transaction logs](#9-latency-percentiles-from-transaction-logs)
10. [Measure cache, I/O and CPU alongside pgbench](#10-measure-cache-io-and-cpu-alongside-pgbench)
11. [Benchmarking methodology](#11-benchmarking-methodology)
12. [Worked example: does shared_buffers matter?](#12-worked-example-does-shared_buffers-matter)
13. [Option reference](#13-option-reference)
14. [Clean up](#14-clean-up)
15. [Common mistakes](#15-common-mistakes)
16. [Version notes](#16-version-notes)
17. [References](#17-references)

---

## 1. What pgbench is (and is not)

`pgbench` runs the same SQL script over and over in several concurrent database sessions and reports how fast the database handled it.

**pgbench reports:**

| Metric | Meaning |
|---|---|
| **TPS** (transactions per second) | Throughput: how many script executions completed per second. |
| **Latency average / stddev** | Responsiveness: how long one transaction took, on average, and how much that varied. |
| **Failed transactions** | Transactions that ended in a serialization or deadlock error and were not retried successfully. |
| **Per-statement latency** (`-r`) | Average time of each statement in the script. |
| **Progress reports** (`-P`) | TPS and latency every N seconds while the test runs. |

**pgbench does not report** buffer cache hit ratio, bytes read or written, I/O operations, or CPU usage. Measure those yourself from PostgreSQL's statistics views and OS tools while the benchmark runs (see [section 10](#10-measure-cache-io-and-cpu-alongside-pgbench)).

By default pgbench runs a workload loosely based on **TPC-B** (it is not a certified TPC-B test, hence the name "TPC-B (sort of)"). You can replace it with your own SQL scripts.

---

## 2. Install pgbench

`pgbench` ships with PostgreSQL. Install PostgreSQL 18 from the official PostgreSQL (PGDG) repositories.

### Ubuntu / Debian

```bash
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh   # adds the PGDG repository
sudo apt install -y postgresql-18
```

Binaries live in `/usr/lib/postgresql/18/bin/`.

### RHEL / Rocky Linux / AlmaLinux 9

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql
sudo dnf install -y postgresql18-server postgresql18-contrib
```

Binaries live in `/usr/pgsql-18/bin/`. The `contrib` package also provides `pg_stat_statements`, which this guide uses.

> CentOS 7 is end-of-life. Use Rocky Linux or AlmaLinux 9 instead.

### Verify

```bash
pgbench --version     # e.g. "pgbench (PostgreSQL) 18.x"
pgbench --help
```

If `pgbench` is not on your `PATH`, call it with the full path or add the bin directory to `PATH` (next section).

---

## 3. Create an isolated lab cluster

Benchmark on a separate cluster so you never load a real database and you control every setting. Run as the `postgres` OS user (or any non-root user).

```bash
# --- paths: pick the line for your OS ---
export PGBIN=/usr/pgsql-18/bin              # RHEL / Rocky / Alma
# export PGBIN=/usr/lib/postgresql/18/bin   # Ubuntu / Debian

export PATH=$PGBIN:$PATH
export PGDATA=$HOME/pgbench_lab/data
export PGPORT=5433                          # avoid clashing with an existing server on 5432

mkdir -p "$PGDATA"
initdb -D "$PGDATA"

cat >> "$PGDATA/postgresql.conf" <<'EOF'

# ---- pgbench lab settings ----
port = 5433
shared_buffers = 256MB                       # change requires restart
max_wal_size = 4GB                           # fewer forced checkpoints during tests
checkpoint_timeout = 15min
log_checkpoints = on
track_io_timing = on                         # I/O timing in pg_stat_* views
shared_preload_libraries = 'pg_stat_statements'
EOF

pg_ctl -D "$PGDATA" -l "$PGDATA/server.log" start
createdb pgbench
psql -d pgbench -c "CREATE EXTENSION IF NOT EXISTS pg_stat_statements;"
```

Notes:

- `shared_buffers` cannot be changed with `SET`. Change it in `postgresql.conf` (or with `ALTER SYSTEM SET shared_buffers = '1GB';`) and **restart**: `pg_ctl -D "$PGDATA" restart`.
- `wal_buffers` defaults to `-1` (automatic: 1/32 of `shared_buffers`, capped at 16 MB), so you usually do not need to set it.
- The `-l` flag of `pg_ctl` is a lowercase **L** (log file), not the digit 1.
- Check a setting with `psql -d pgbench -c "SHOW shared_buffers;"`.

---

## 4. Initialize the test database

```bash
pgbench -i -s 50 pgbench
```

`-i` creates and fills four tables. `-s` (scale factor) multiplies the number of rows:

| Table | Rows at scale 1 | Rows at scale 50 |
|---|---:|---:|
| `pgbench_branches` | 1 | 50 |
| `pgbench_tellers` | 10 | 500 |
| `pgbench_accounts` | 100,000 | 5,000,000 |
| `pgbench_history` | 0 | 0 (grows during the test) |

Each scale unit is roughly **15–16 MB** of data, so scale 50 is about 750–800 MB.

### Choose a scale

Pick the scale based on what you want to test:

| Goal | Data size | Example (server with 16 GB RAM, `shared_buffers` = 4 GB) |
|---|---|---|
| CPU and locking, no disk reads | Fits in `shared_buffers` | scale ≈ 100 (~1.5 GB) |
| OS page cache behaviour | Bigger than `shared_buffers`, smaller than RAM | scale ≈ 500 (~7.5 GB) |
| Real disk I/O | Bigger than RAM | scale ≈ 2,000 (~30 GB) |

Also keep **scale ≥ number of clients** for the default test. With fewer branches than clients, every client fights over the same `pgbench_branches` rows and you measure lock contention, not performance.

### Useful initialization options

```bash
# Remote server
pgbench -i -s 50 -h 10.10.1.200 -p 5432 -U postgres pgbench

# Leave 10% free space per page (fillfactor 90) so updates can stay HOT
pgbench -i -s 50 -F 90 pgbench

# Generate data on the server (faster over a network; no progress messages)
pgbench -i -s 500 -I dtGvp pgbench

# Partition pgbench_accounts into 8 hash partitions
pgbench -i -s 500 --partitions=8 --partition-method=hash pgbench

# Add foreign keys between the tables
pgbench -i -s 50 --foreign-keys pgbench

# Fewer progress lines for big scales (one every 5 seconds)
pgbench -i -s 1000 -q pgbench
```

`-F` is the **fillfactor** (10–100, default 100), the percentage of each page filled during load. A lower value makes the tables larger and leaves room for HOT updates. It is not a "size" setting and has nothing to do with the scale factor.

`-I` runs initialization steps in the order given. Default is `dtgvp`:

| Step | Action |
|---|---|
| `d` | Drop existing pgbench tables |
| `t` | Create tables |
| `g` | Generate data on the client (`COPY`) |
| `G` | Generate data on the server |
| `v` | `VACUUM` the tables |
| `p` | Create primary keys |
| `f` | Create foreign keys (not run by default) |

### Check the result

Exact row counts:

```sql
\c pgbench
SELECT 'pgbench_branches' AS table_name, count(*) FROM pgbench_branches
UNION ALL SELECT 'pgbench_tellers',  count(*) FROM pgbench_tellers
UNION ALL SELECT 'pgbench_accounts', count(*) FROM pgbench_accounts
UNION ALL SELECT 'pgbench_history',  count(*) FROM pgbench_history;
```

Table sizes:

```sql
SELECT relname AS table_name,
       pg_size_pretty(pg_total_relation_size(oid)) AS total_size
FROM pg_class
WHERE relname LIKE 'pgbench_%' AND relkind IN ('r', 'p')
ORDER BY pg_total_relation_size(oid) DESC;
```

Or from `psql`: `\dt+ pgbench_*`.

Exact row counts for **every** table in a database (slow on big databases, because each `count(*)` scans the table):

```sql
CREATE OR REPLACE FUNCTION count_rows(p_schema text, p_table text)
RETURNS bigint
LANGUAGE plpgsql AS
$$
DECLARE
  result bigint;
BEGIN
  EXECUTE format('SELECT count(*) FROM %I.%I', p_schema, p_table) INTO result;
  RETURN result;
END;
$$;

SELECT table_schema, table_name, count_rows(table_schema, table_name) AS row_count
FROM information_schema.tables
WHERE table_schema NOT IN ('pg_catalog', 'information_schema')
  AND table_type = 'BASE TABLE'
ORDER BY row_count DESC;
```

`format('%I')` quotes identifiers safely, so mixed-case or unusual table names work and cannot inject SQL.

---

## 5. Run a benchmark

### Baseline

```bash
pgbench -c 10 -j 2 -T 300 -P 30 -r pgbench
```

| Flag | Meaning |
|---|---|
| `-c 10` | 10 concurrent client sessions |
| `-j 2` | 2 pgbench worker threads driving those clients (keep `-j` ≤ CPU cores of the machine running pgbench; `-c` should be a multiple of `-j`) |
| `-T 300` | Run for 300 seconds (use `-t N` instead for N transactions **per client**) |
| `-P 30` | Print a progress line every 30 seconds |
| `-r` | Report average latency per statement at the end |

With `-t`, the total number of transactions is **clients × transactions per client**. For example, `-c 50 -t 100` runs 50 × 100 = **5,000** transactions in total.

Prefer `-T` (time-based) for real benchmarks: every run lasts long enough to include checkpoints and autovacuum activity.

### More examples

```bash
# Remote server
pgbench -h 10.10.1.200 -p 5432 -U postgres -c 10 -j 2 -T 300 pgbench

# Read-only workload (one SELECT per transaction)
pgbench -S -c 32 -j 8 -T 300 -P 30 pgbench

# Fewer updates (skips the tellers and branches updates; less row contention)
pgbench -N -c 32 -j 8 -T 300 pgbench

# Prepared statements (removes parse/plan cost; closer to most application drivers)
pgbench -M prepared -c 32 -j 8 -T 300 pgbench

# Fixed load instead of maximum load: 500 TPS target, count transactions slower than 50 ms
pgbench -R 500 -L 50 -c 32 -j 8 -T 300 -P 30 pgbench

# New connection for every transaction (measures connection overhead, e.g. with/without PgBouncer)
pgbench -C -c 32 -j 8 -T 120 pgbench

# Serializable isolation with automatic retries of serialization/deadlock failures
PGOPTIONS='-c default_transaction_isolation=serializable' \
  pgbench --max-tries=10 --failures-detailed -c 32 -j 8 -T 300 pgbench
```

`-R` (rate) is the most realistic mode for latency testing: real applications send a steady stream of requests rather than hammering the database as fast as possible. Without `-R`, pgbench measures **maximum throughput**, and latency mostly reflects queueing.

Before each run pgbench vacuums the standard tables (`starting vacuum...end.`). Use `-n` to skip that, which is required when a custom script doesn't use the pgbench tables.

---

## 6. Read the output

Typical output from PostgreSQL 18:

```text
pgbench (18.x)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
maximum number of tries: 1
duration: 300 s
number of transactions actually processed: 268412
number of failed transactions: 0 (0.000%)
latency average = 11.176 ms
latency stddev = 6.902 ms
initial connection time = 21.458 ms
tps = 894.698203 (without initial connection time)
```

| Line | What it tells you |
|---|---|
| `transaction type` | Which script ran |
| `scaling factor` | Data size (read from the tables for built-in scripts) |
| `query mode` | `simple`, `extended` or `prepared` (`-M`) |
| `number of clients` / `threads` | `-c` / `-j` |
| `maximum number of tries` | `--max-tries` (1 = no retries) |
| `duration` or `number of transactions per client` | `-T` or `-t` |
| `number of transactions actually processed` | Completed transactions. With `-t` it shows `done/expected`, and a mismatch means some clients failed. |
| `number of failed transactions` | Serialization or deadlock failures that weren't retried successfully |
| `latency average` / `stddev` | Mean and spread of transaction time. A large stddev means unstable performance (check for checkpoints or lock waits). |
| `initial connection time` | Time to open the connections, reported separately |
| `tps` | Throughput, **excluding** connection time |

Extra lines appear with some options:

- `-r`: a `statement latencies in milliseconds and failures` block (average time per statement).
- `-L`: `number of transactions above the 50.0 ms latency limit`.
- `-R`: `rate limit schedule lag` (how far behind the schedule pgbench fell). If the lag keeps growing, the server can't sustain the target rate.
- `--max-tries` > 1: `number of transactions retried` and `total number of retries`.

Older PostgreSQL versions (13 and earlier) printed two TPS lines, "including connections establishing" and "excluding connections establishing". PostgreSQL 14 and later print one TPS value plus `initial connection time`.

A progress line from `-P` looks like:

```text
progress: 30.0 s, 902.3 tps, lat 11.07 ms stddev 6.81, 0 failed
```

---

## 7. Built-in scripts

| Name | Shortcut | What it does |
|---|---|---|
| `tpcb-like` | (default) | 3 updates, 1 select, 1 insert per transaction |
| `simple-update` | `-N` | Like tpcb-like but without the tellers and branches updates |
| `select-only` | `-S` | A single primary-key `SELECT` |

```bash
pgbench -b list                         # list built-in scripts
pgbench --show-script=tpcb-like         # print a script's text
pgbench -b tpcb-like@1 -b select-only@9 -c 16 -j 4 -T 300 pgbench   # mix: 90% reads, 10% writes
```

The default `tpcb-like` script:

```sql
\set aid random(1, 100000 * :scale)
\set bid random(1, 1 * :scale)
\set tid random(1, 10 * :scale)
\set delta random(-5000, 5000)
BEGIN;
UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
UPDATE pgbench_tellers SET tbalance = tbalance + :delta WHERE tid = :tid;
UPDATE pgbench_branches SET bbalance = bbalance + :delta WHERE bid = :bid;
INSERT INTO pgbench_history (tid, bid, aid, delta, mtime) VALUES (:tid, :bid, :aid, :delta, CURRENT_TIMESTAMP);
END;
```

One **transaction** in pgbench means one complete run of the script: here 7 SQL commands, from `BEGIN` to `END`. When several scripts are given, each transaction picks one at random according to its `@weight` (default 1; weight 0 disables a script).

---

## 8. Custom scripts for realistic workloads

The built-in test rarely looks like your application. For realistic numbers, write scripts based on your real top queries (for example, from `pg_stat_statements`), weighted by how often each one runs.

### Script syntax

| Command | Purpose |
|---|---|
| `\set name expression` | Set a variable (`random()`, arithmetic, etc.) |
| `:name` | Use a variable inside SQL |
| `\sleep 10 ms` | Pause (simulate user think time) |
| `\if` / `\elif` / `\else` / `\endif` | Conditional logic |
| `\gset` (after a query) | Store the query's result columns into variables |
| `\startpipeline` / `\endpipeline` | Send statements in pipeline mode (requires `-M extended` or `-M prepared`) |
| `\setshell`, `\shell` | Run a shell command |

Random distributions, useful for realistic skew:

| Function | Behaviour |
|---|---|
| `random(lb, ub)` | Uniform |
| `random_gaussian(lb, ub, parameter)` | Normal distribution around the middle |
| `random_exponential(lb, ub, parameter)` | Low values far more frequent |
| `random_zipfian(lb, ub, parameter)` | "Hot rows": a few keys get most of the traffic (parameter > 1.0) |

Variables available automatically: `:scale` and `:client_id`. Define your own with `-D name=value`.

### Example: a mixed read/write workload with hot rows

`read_account.sql`:

```sql
\set aid random_zipfian(1, 100000 * :scale, 1.07)
SELECT aid, abalance FROM pgbench_accounts WHERE aid = :aid;
```

`range_report.sql` (a list-view-style query: a range, sorted, limited):

```sql
\set start random(1, 100000 * :scale - 1000)
SELECT aid, abalance
FROM pgbench_accounts
WHERE aid BETWEEN :start AND :start + 1000
ORDER BY abalance DESC
LIMIT 50;
```

`update_account.sql`:

```sql
\set aid random_zipfian(1, 100000 * :scale, 1.07)
\set delta random(-5000, 5000)
BEGIN;
UPDATE pgbench_accounts SET abalance = abalance + :delta WHERE aid = :aid;
INSERT INTO pgbench_history (tid, bid, aid, delta, mtime)
VALUES (1, 1, :aid, :delta, CURRENT_TIMESTAMP);
END;
```

Run them as 70% point reads, 10% range reports, 20% updates:

```bash
pgbench -n -s 50 -M prepared -c 32 -j 8 -T 600 -P 30 -r \
  -f read_account.sql@7 \
  -f range_report.sql@1 \
  -f update_account.sql@2 \
  pgbench
```

- `-n`: skip the automatic vacuum (good practice with custom scripts).
- `-s 50`: with custom scripts, pgbench doesn't detect the scale on its own. Pass the scale you initialized with, so `:scale` and the report are correct.
- `-r`: shows which statement is slow.

### Build scripts from production queries

```sql
SELECT calls,
       round(mean_exec_time::numeric, 2) AS mean_ms,
       round(total_exec_time::numeric)   AS total_ms,
       left(query, 120) AS query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

Use the top statements and their `calls` ratio as script weights, on a copy of production-sized data with the same distribution.

---

## 9. Latency percentiles from transaction logs

Average latency hides outliers. Users feel p95 and p99. Log every transaction and compute percentiles:

```bash
pgbench -l --log-prefix=run1 -c 32 -j 8 -T 300 pgbench
```

This writes `run1.<pid>` (and `run1.<pid>.<thread>` for extra threads). Each line looks like:

```text
client_id  transaction_no  time_us  script_no  time_epoch  time_epoch_us  [schedule_lag]  [retries]
```

The third column is the transaction latency in **microseconds**. Failed or skipped transactions show `failed` or `skipped` there instead of a number.

```bash
cat run1.* \
  | awk '$3 ~ /^[0-9]+$/ {print $3}' \
  | sort -n \
  | awk '{a[NR]=$1}
         END {
           printf "transactions: %d\n", NR
           printf "p50: %.2f ms\n", a[int(NR*0.50)]/1000
           printf "p95: %.2f ms\n", a[int(NR*0.95)]/1000
           printf "p99: %.2f ms\n", a[int(NR*0.99)]/1000
           printf "max: %.2f ms\n", a[NR]/1000
         }'
```

For long runs, log a sample, or log one summary line per time interval:

```bash
pgbench -l --sampling-rate=0.05 -c 32 -j 8 -T 3600 pgbench          # 5% of transactions
pgbench -l --aggregate-interval=10 -c 32 -j 8 -T 3600 pgbench        # one line per 10 s
```

---

## 10. Measure cache, I/O and CPU alongside pgbench

pgbench measures the client's view. Collect the server's view at the same time.

### Buffer cache hit ratio (PostgreSQL statistics)

Reset the counters just before the run, then read them just after:

```sql
-- before the run (resets statistics for the current database)
SELECT pg_stat_reset();
SELECT pg_stat_statements_reset();

-- after the run
SELECT datname,
       blks_hit,
       blks_read,
       round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 2) AS cache_hit_pct,
       xact_commit,
       xact_rollback,
       temp_bytes,
       deadlocks
FROM pg_stat_database
WHERE datname = 'pgbench';
```

`blks_read` counts blocks PostgreSQL requested from the OS. Some of those come from the OS page cache, not the disk, so a "miss" here isn't always a physical read.

### I/O by backend type (PostgreSQL 16+)

```sql
SELECT backend_type, object, context, reads, writes, extends, hits
FROM pg_stat_io
WHERE coalesce(reads, 0) + coalesce(writes, 0) + coalesce(hits, 0) > 0
ORDER BY reads DESC NULLS LAST;
```

If `client backend` shows many `writes`, sessions are writing dirty buffers themselves. That usually means `shared_buffers` is too small or the background writer can't keep up.

### Checkpoints and WAL

With `log_checkpoints = on`, check the server log for how often checkpoints ran during the test and how long they took. Frequent checkpoints triggered by WAL volume mean `max_wal_size` is too small for the workload.

### Operating system (run in another terminal during the test)

```bash
vmstat 5                 # run queue (r), blocked (b), swap (si/so), CPU split
iostat -x 5              # per-disk r_await / w_await, queue size, utilization
pidstat -u -d 5          # CPU and disk I/O per process (postgres backends vs pgbench)
sar -n DEV 5             # network throughput (remote benchmarks)
```

If the `pgbench` process itself reaches 100% CPU, the **client** is the bottleneck. Add threads (`-j`) or run pgbench from a separate machine.

---

## 11. Benchmarking methodology

1. **Write down the question first.** For example: "Does raising `shared_buffers` from 256 MB to 2 GB improve read throughput at scale 500?"
2. **Change one variable at a time.** Keep scale, clients, threads, duration, script and hardware identical between runs.
3. **Reinitialize or restore between runs.** Updates bloat the tables and `pgbench_history` keeps growing, so later runs get slower. Re-run `pgbench -i`, or restore a snapshot, before each comparison.
4. **Warm up.** Run a few minutes first and discard the result, or make runs long enough (10+ minutes) that cold-cache effects are small.
5. **Run each test at least 3 times** and report the median plus the spread.
6. **Include checkpoints.** Runs shorter than `checkpoint_timeout` can look better than reality.
7. **Separate client and server.** For serious tests, run pgbench on another machine so it doesn't steal CPU from PostgreSQL.
8. **Record everything:** PostgreSQL version, key settings (`SHOW ALL`), hardware, OS, scale, exact command line.
9. **Test latency at a realistic rate (`-R`),** not only maximum throughput.
10. **Never benchmark a production server** without explicit approval. pgbench generates heavy load and writes data.

### Results template

| Run | Setting changed | Scale | Clients/threads | Duration | TPS | Avg latency (ms) | p95 (ms) | p99 (ms) | Cache hit % | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | baseline | 500 | 32/8 | 600 s | | | | | | |
| 2 | | 500 | 32/8 | 600 s | | | | | | |
| 3 | | 500 | 32/8 | 600 s | | | | | | |

---

## 12. Worked example: does shared_buffers matter?

```bash
# 1. Initialize a data set bigger than the small setting but smaller than RAM
pgbench -i -s 300 -q pgbench                      # ~4.5 GB

# 2. Run A: shared_buffers = 256MB
psql -d pgbench -c "ALTER SYSTEM SET shared_buffers = '256MB';"
pg_ctl -D "$PGDATA" restart
pgbench -S -T 120 -c 16 -j 4 pgbench > /dev/null  # warm-up, discarded
psql -d pgbench -c "SELECT pg_stat_reset();"
pgbench -S -c 16 -j 4 -T 600 -P 60 pgbench | tee run_a.txt
psql -d pgbench -c "SELECT round(100.0*blks_hit/nullif(blks_hit+blks_read,0),2) AS hit_pct FROM pg_stat_database WHERE datname='pgbench';"

# 3. Run B: shared_buffers = 4GB (same steps)
psql -d pgbench -c "ALTER SYSTEM SET shared_buffers = '4GB';"
pg_ctl -D "$PGDATA" restart
pgbench -S -T 120 -c 16 -j 4 pgbench > /dev/null
psql -d pgbench -c "SELECT pg_stat_reset();"
pgbench -S -c 16 -j 4 -T 600 -P 60 pgbench | tee run_b.txt
psql -d pgbench -c "SELECT round(100.0*blks_hit/nullif(blks_hit+blks_read,0),2) AS hit_pct FROM pg_stat_database WHERE datname='pgbench';"

# 4. Compare
grep -E "^tps|latency average" run_a.txt run_b.txt
```

This test is read-only (`-S`), so the tables don't change between runs and reinitializing isn't needed. Repeat each run three times before drawing conclusions.

---

## 13. Option reference

### Initialization (`pgbench -i [options] dbname`)

| Option | Meaning |
|---|---|
| `-i`, `--initialize` | Initialization mode |
| `-s N`, `--scale=N` | Scale factor (default 1). At scale ≥ 20,000 the account IDs become `bigint`. |
| `-F N`, `--fillfactor=N` | Table fillfactor, 10–100 (default 100) |
| `-I steps`, `--init-steps=steps` | Steps to run: `d t g G v p f` (default `dtgvp`) |
| `-n`, `--no-vacuum` | Skip vacuum during initialization |
| `-q`, `--quiet` | Progress message every 5 s instead of every 100,000 rows |
| `--foreign-keys` | Add foreign keys |
| `--partitions=N` | Partition `pgbench_accounts` into N partitions |
| `--partition-method=range\|hash` | Partitioning method (default `range`) |
| `--tablespace=TS`, `--index-tablespace=TS` | Place tables or indexes in a tablespace |
| `--unlogged-tables` | Create unlogged tables (no WAL; data lost on crash) |

### Benchmarking (`pgbench [options] dbname`)

| Option | Meaning |
|---|---|
| `-c N`, `--client=N` | Concurrent sessions (default 1) |
| `-j N`, `--jobs=N` | pgbench worker threads (default 1) |
| `-t N`, `--transactions=N` | Transactions **per client** (default 10) |
| `-T S`, `--time=S` | Run for S seconds (cannot combine with `-t`) |
| `-b name[@w]`, `--builtin=name[@w]` | Built-in script with optional weight |
| `-f file[@w]`, `--file=file[@w]` | Custom script with optional weight |
| `-S`, `--select-only` | Same as `-b select-only` |
| `-N`, `--skip-some-updates` | Same as `-b simple-update` |
| `-M mode`, `--protocol=mode` | `simple` (default), `extended`, `prepared` |
| `-n`, `--no-vacuum` | Skip pre-run vacuum |
| `-v`, `--vacuum-all` | Vacuum all four tables before the run |
| `-C`, `--connect` | New connection per transaction |
| `-D var=value`, `--define=var=value` | Define a script variable |
| `-s N`, `--scale=N` | Scale value to report and use as `:scale` with custom scripts |
| `-P S`, `--progress=S` | Progress report every S seconds |
| `--progress-timestamp` | Unix timestamps in progress lines |
| `-r`, `--report-per-command` | Per-statement latency, failures and retries |
| `-R TPS`, `--rate=TPS` | Target rate instead of maximum speed |
| `-L ms`, `--latency-limit=ms` | Count, or skip with `-R`, transactions slower than the limit |
| `-l`, `--log` | Per-transaction log file |
| `--log-prefix=name` | Log file name prefix (default `pgbench_log`) |
| `--aggregate-interval=S` | One aggregated log line per S seconds |
| `--sampling-rate=fraction` | Log only a fraction of transactions |
| `--max-tries=N` | Retry serialization/deadlock failures up to N tries (default 1; 0 = unlimited, needs `-T` or `-L`) |
| `--failures-detailed` | Split failures into serialization and deadlock |
| `--verbose-errors` | Print every error message |
| `--exit-on-abort` | Stop the whole run if any client aborts |
| `--random-seed=time\|rand\|N` | Seed for reproducible random values |
| `--show-script=name` | Print a built-in script and exit |

### Connection and general

| Option | Meaning |
|---|---|
| `-h host`, `-p port`, `-U user` | Connection settings (also `PGHOST`, `PGPORT`, `PGUSER`) |
| `-d dbname` | Database (or give it as the last argument; default `PGDATABASE`) |
| `-V`, `--version` | Show version |
| `-?`, `--help` | Show help |
| `--debug` | Debug output |

### Exit status

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Invalid options or startup error |
| 2 | Error during the run (partial results are still printed unless `--exit-on-abort` is used) |

---

## 14. Clean up

```bash
pgbench -i -I d pgbench          # drop only the pgbench tables
dropdb pgbench                   # or drop the whole database
pg_ctl -D "$PGDATA" stop         # stop the lab cluster
rm -rf "$HOME/pgbench_lab"       # remove the lab cluster entirely (irreversible)
```

---

## 15. Common mistakes

| Mistake | Correct |
|---|---|
| `psql -i ...` to initialize | `pgbench -i ...`. `-i` is a pgbench option. |
| `SET shared_buffers = '256MB';` | Not possible at session level. Set it in `postgresql.conf` or with `ALTER SYSTEM`, then restart. |
| `CREATEDB pgbench` / `DROPDB pgbench` in psql | `createdb pgbench` / `dropdb pgbench` are shell commands. In psql use `CREATE DATABASE pgbench;`. |
| `pg_ctl start -1 logfile` | `pg_ctl -D "$PGDATA" -l logfile start`. Lowercase L. |
| "Use `-F` bigger than the scale factor to grow the database" | `-F` is fillfactor (10–100%). Size comes from `-s`. |
| "`-c 50 -t 100` = 50,000 transactions" | It's 50 × 100 = **5,000** in total (`-t` is per client). |
| Expecting pgbench to report cache hit ratio, I/O or CPU | Measure these with `pg_stat_database`, `pg_stat_io` and OS tools during the run. |
| Scale smaller than the number of clients | Clients collide on the same branch rows. Use scale ≥ clients. |
| 30-second runs | Too short. Use `-T` of several minutes so checkpoints and autovacuum are included. |
| Comparing runs without reinitializing | Bloat and `pgbench_history` growth make later runs slower. Reinitialize between comparisons. |
| Running pgbench on the same small server it tests | pgbench competes for CPU. Use a separate client machine for serious tests. |
| Building the benchmark on CentOS 7 | End-of-life. Use Rocky/Alma 9, RHEL 9 or Ubuntu LTS. |

---

## 16. Version notes

Changes worth knowing if you compare with older guides:

- **PostgreSQL 13:** server-side data generation (`-I G`) and partitioned `pgbench_accounts` (`--partitions`, `--partition-method`).
- **PostgreSQL 14:** output shows one `tps` line plus `initial connection time`, replacing the old "including/excluding connections establishing" lines.
- **PostgreSQL 15:** retries and failure reporting: `--max-tries`, `--failures-detailed`, `--verbose-errors`, and the "number of failed transactions" line.
- **PostgreSQL 17:** `--exit-on-abort`.
- **PostgreSQL 18:** current release. Options and output as shown in this guide.

Always check `pgbench --help` on your installed version.

---

## 17. References

- [pgbench documentation (PostgreSQL 18)](https://www.postgresql.org/docs/current/pgbench.html)
- [PostgreSQL downloads and PGDG repositories](https://www.postgresql.org/download/)
- [pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html)
- [Cumulative statistics views (`pg_stat_database`, `pg_stat_io`)](https://www.postgresql.org/docs/current/monitoring-stats.html)
- [Resource consumption settings (`shared_buffers`, `work_mem`)](https://www.postgresql.org/docs/current/runtime-config-resource.html)
