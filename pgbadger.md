```shell
ubuntu@host-machine:~$ sudo apt install perl perl-devel -y
ubuntu@host-machine:~$ wget https://github.com/darold/pgbadger/archive/v11.4.tar.gz
ubuntu@host-machine:~$ tar xzvf v11.4.tar.gz
ubuntu@host-machine:~$ cd pgbadger-11.4/
ubuntu@host-machine:~/pgbadger-11.4$ perl Makefile.PL
Checking if your kit is complete...
Looks good
Generating a Unix-style Makefile
Writing Makefile for pgBadger
Writing MYMETA.yml and MYMETA.json
```

```shell
ubuntu@host-machine:~/pgbadger-11.4$ make && sudo make install
cp pgbadger blib/script/pgbadger
"/usr/bin/perl" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/pgbadger
echo "=head1 SYNOPSIS" > doc/synopsis.pod
./pgbadger --help >> doc/synopsis.pod
echo "=head1 DESCRIPTION" >> doc/synopsis.pod
sed -i.bak 's/ +$//g' doc/synopsis.pod
rm doc/synopsis.pod.bak
sed -i.bak '/^=head1 SYNOPSIS/,/^=head1 DESCRIPTION/d' doc/pgBadger.pod
sed -i.bak '4r doc/synopsis.pod' doc/pgBadger.pod
rm doc/pgBadger.pod.bak
Manifying 1 pod document
rm doc/synopsis.pod
echo "=head1 SYNOPSIS" > doc/synopsis.pod
./pgbadger --help >> doc/synopsis.pod
echo "=head1 DESCRIPTION" >> doc/synopsis.pod
sed -i.bak 's/ +$//g' doc/synopsis.pod
rm doc/synopsis.pod.bak
sed -i.bak '/^=head1 SYNOPSIS/,/^=head1 DESCRIPTION/d' doc/pgBadger.pod
sed -i.bak '4r doc/synopsis.pod' doc/pgBadger.pod
rm doc/pgBadger.pod.bak
Manifying 1 pod document
Installing /usr/local/man/man1/pgbadger.1p
Installing /usr/local/bin/pgbadger
Appending installation info to /usr/local/lib/x86_64-linux-gnu/perl/5.30.0/perllocal.pod
rm doc/synopsis.pod
```
```shell
ubuntu@host-machine:~/pgbadger-11.4$ pgbadger -V
ubuntu@host-machine:$ sudo vim /etc/postgresql/14/main/postgresql.conf


log_min_duration_statement = 0
log_line_prefix = '%t [%p]: user=%u,db=%d,app=%a,client=%h '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
log_autovacuum_min_duration = 0
log_error_verbosity = default
lc_messages='en_US.UTF-8'
```
```shell
ubuntu@host-machine:$ sudo systemctl restart postgresql
ubuntu@host-machine:$ sudo pgbadger /var/log/postgresql/postgresql-14-main.log
ubuntu@host-machine:$ scp /var/log/postgresql/out.html maly@172.16.4.74:~



    pgbadger /var/log/postgresql.log
    pgbadger /var/log/postgres.log.2.gz /var/log/postgres.log.1.gz /var/log/postgres.log
    pgbadger /var/log/postgresql/postgresql-2012-05-*
    pgbadger --exclude-query="^(COPY|COMMIT)" /var/log/postgresql.log
    pgbadger -b "2012-06-25 10:56:11" -e "2012-06-25 10:59:11" /var/log/postgresql.log
    cat /var/log/postgres.log | pgbadger -
    # Log prefix with stderr log output
    pgbadger --prefix '%t [%p]: user=%u,db=%d,client=%h' /pglog/postgresql-2012-08-21*
    pgbadger --prefix '%m %u@%d %p %r %a : ' /pglog/postgresql.log
    # Log line prefix with syslog log output
    pgbadger --prefix 'user=%u,db=%d,client=%h,appname=%a' /pglog/postgresql-2012-08-21*
    # Use my 8 CPUs to parse my 10GB file faster, much faster
    pgbadger -j 8 /pglog/postgresql-10.1-main.log
```





