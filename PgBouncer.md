   
```shell
# update package List 
sudo apt-get update -y 

# install pgbouncer
sudo apt-get install pgbouncer -y


# permissions change to postgres user
sudo su
chown postgres:postgres /etc/pgbouncer/ -R
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

# benchmark testing
pgbench -h 172.16.7.90 -p 6432 -c 10 -t 10000 -U testuser -d testdb
pgbench -h 172.16.7.90 -p 5432 -c 10 -t 10000 -U testuser -d testdb
```

