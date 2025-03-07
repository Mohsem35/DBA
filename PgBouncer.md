   
```shell
# update package List 
sudo apt-get update -y 

# install pgbouncer
sudo apt-get install pgbouncer -y


# permissions change to postgres user
sudo su
chown postgres:postgres /etc/pgbouncer/ -R
usermod -a -G sudo postgres
sudo su - postgres
cd /etc/pgbouncer/

# make a copy of config file
cp pgbouncer.ini pgbouncer.ini.copy

# open pgbouncer config file
vim /etc/pgbouncer/pgbouncer.ini

# change the following parametes of pgbouncer config file

[databases]
testdb = host=172.16.7.90 port=5432 dbname=testdb


[pgbouncer]
logfile = /var/log/postgresql/pgbouncer.log
pidfile = /var/run/postgresql/pgbouncer.pid
listen_addr = *
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = session
max_client_conn = 100
default_pool_size = 20
reserve_pool_size = 25

# edit authentication file

vim /etc/pgbouncer/userlist.txt

# add the line
"testdb" "Password"


# starting pgbouncer
sudo systemctl enable pgbouncer --now
sudo systemctl restart pgbouncer
sudo systemctl status pgbouncer.service

# connect to database through pgbouncer
psql -h 127.0.0.1 -p 6432 -U testdb

psql -h 118.67.215.244 -p 6432 -U your_user -d your_database

> this command attempts to connect to your database through PgBouncer

# benchmark testing
pgbench -h 172.16.7.90 -p 6432 -c 10 -t 10000 -U testuser -d testdb
pgbench -h 172.16.7.90 -p 5432 -c 10 -t 10000 -U testuser -d testdb
```

Refferences:

- [Efficient PostgreSQL Management: A Complete Guide to Installing and Configuring ](https://www.linkedin.com/pulse/efficient-postgresql-management-complete-guide-installing-configuring-rxkzc/)
- [PgBouncer Installation, Configuration and Use Cases for Better Performance](https://medium.com/swlh/pgbouncer-installation-configuration-and-use-cases-for-better-performance-1806316f3a22)
- [PGBouncer Installation on Linux](https://help.datagaps.com/articles/#!etl-validator/pgbouncer-installation-on-linux/a/h2_812968334)
- [Installing PgBouncer on Ubuntu/Debian](https://www.scaleway.com/en/docs/tutorials/install-pgbouncer/)
- [Setup PostgreSQL Connection Pooling with PgBouncer](https://medium.com/@sozcandba/setup-postgresql-connection-pooling-with-pgbouncer-eb160bb0ca0a)
