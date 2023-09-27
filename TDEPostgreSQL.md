
Make sure that all the dependencies are met.

- _GNU make version 3.81 or newer is required_; other make programs or older GNU make versions will not work. (GNU make is sometimes installed under the name gmake.) To test for GNU make enter:

```
make --version
```
- You need an _ISO/ANSI C compiler_ (at least C99-compliant). Recent versions of GCC are recommended, but PostgreSQL is known to build using a wide variety of compilers from different vendors.
```
gcc --version
```
- _tar is required to unpack the source distribution_, in addition to either gzip or bzip2.
- The _GNU Readline library_ is used by default. It allows psql (the PostgreSQL command line SQL interpreter) to remember each command you type, and allows you to use arrow keys to recall and edit previous commands.

```
ldconfig -p | grep readline
```

- The _zlib compression library_ is used by default
```
dpkg -l | grep zlib1g
```
- The _ICU locale provider_ is used by default. Using this option disables support for ICU collation features (see Section 24.2).ICU support requires the ICU4C package to be installed. The minimum required version of ICU4C is currently 4.2.
```
dpkg -l | grep icu
```


### 1. Download the source code
To download PostgreSQL TDE, you can go to our ​website1 and download the latest tar file. If you prefer to use command line, you can also simply use “wget” to get the file. Here is an example:

```
wget https://download.cybertec-postgresql.com/postgresql-12.3_TDE_1.0.tar.gz
```

### 2. Extracting and configuring PostgreSQL 12 TDE

Once you have downloaded the file, you can easily unpack it. Here is how it works:
```
tar xvfz postgresql-12.3_TDE_1.0.tar.gz
```
##### For Linux:

On Linux, compiling PostgreSQL is pretty straight forward. Make sure that all the dependencies outlined in the documentation are met. A directory will be created containing the entire source code of PostgreSQL TDE. Enter
the directory and execute the following command:

```
cd postgresql-12.3_TDE_1.0/
```

```
sudo apt-get update
sudo apt-get install build-essential checkinstall zlib1g-dev postgresql-client-common libreadline-dev libpq-dev
sudo apt-get install libssl-dev postgresql-common
```

```
sudo apt-get install libldap2-dev
sudo apt-get install libperl-dev
sudo apt-get install flex
sudo apt-get install bison
sudo apt-get install zlibc zlib1g-dev openssl python-dev
```


```
# in pgsql directory all files will be installed
./configure --prefix=/usr/local/pgsql --with-openssl --with-perl \
   --with-python --with-ldap
```

If something goes wrong during configuring, it is very likely that you have missed a package. 

### 3. Compiling the code

Once the “configure” step has been executed successfully, you can easily compile the code:

```
sudo make install
cd contrib
sudo make install
```

If you want to use more than one CPU core, you can add the -j flag to “make” to compile code in parallel. 

PostgreSQL is now ready, and you can already prepare your server infrastructure by adjusting the $PATH environment variables so you can easily access the database.

```
ls -lh /usr/local/pgsql/
```
⎟   Note: If you see (`bin` `include` `lib` `share`) means we have installed the customized postgresql


### 4. Setting up key management

Key management is an important aspect. To encrypt a database instance, a key to do so has to come from somewhere. In case of PostgreSQL TDE, the key is coming from an flexible external program. Ideally the key DOES NOT COME from the local filesystem but from remote secure keystore.

Before creating your database instance, you have to write some code to make sure that the key can be read by the database during startup and instance creation.

Here is the most simplistic example possible:

```
cat /somewhere/provide_key.sh 
```

```
#!/bin/sh 
echo 882fb7c12e80280fd664c69d2d636913
```

All you need is a program that prints the key to stdout – and that’s it! Make sure that PostgreSQL is able to execute this program:
```
chmod +x /somewhere/provide_key.sh
```

⎟   Note: You don’t have to write a shell script – you can use any kind of executable such as a C, Go or Python.


### 5. Creating a database instance / cluster


Now log in as postgres user:
```
sudo su - postgres
```
After this step, you now need to initialize your database in a directory of your choice. This is essential to be on the safe side, that is to reduce any permission errors during the running of the postgres server. 
Create the directory and add the attributes to it for access later on:

```
sudo mkdir /usr/local/postgres
sudo chmod 775 /usr/local/postgres
sudo chown postgres /usr/local/postgres
```

Now you can initialize your database with the key you created in the provide_key.sh file above. Before that, you also need to set the export path for the database

    export PATH=$PATH:/usr/local/pgsql/bin

And finally hit:

    initdb -D /usr/local/postgres -K /provide_key.sh

This will enable the encryption and provide you with a command at the very end to copy and run for executing the encrypted server. The command will look something like this:


```
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "C.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.
Data encryption is enabled.

fixing permissions on existing directory /usr/local/postgres ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Asia/Dhaka
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D /usr/local/postgres -l logfile start

```
### Database log configuration

```
# create custom log directory
sudo su - postgres
sudo mkdir /usr/local/pgsql/data/pg_log/
sudo chmod 777 /usr/local/pgsql/data/pg_log
```

Edit the **`postgresql.conf`** file for this custom logging

```
logging_collector = on
log_directory = /usr/local/pgsql/data/pg_log
log_filename = <uncomment_this_value>
```

```
sudo systemctl start postgresql@14-initdb.service
```

```
export PATH=$PATH:/usr/bin
```
```
# data directory
/var/lib/postgresql/14/main
```
```
sudo /usr/bin/pg_createcluster 14 initdb -D /usr/local/postgres -K /home/ubuntu/postgresql-12.3_TDE_1.0/somewhere/provide_key.sh
```
