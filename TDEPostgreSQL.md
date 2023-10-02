




new database/cluster cncrypt করার way পাওয়া গেছে, কিন্তু existing database কিভাবে encrypt করব তার way পাই নাই। 



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




## Encrypting Database using Transparent Data Encryption (TDE)

### Prerequisites

To complete this , you will need:

An **`Ubuntu 22.10 server`** configured by following our [Initial Server Setup for Ubuntu 20.04 guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04) and then setting up a
[PostgreSQL installation](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart)
    
Installation of the following libraries
    
- Bison
- ReadLine
- Flex
- Zlib
- OpenSSL
- Crypto

```
sudo apt-get install libreadline8 libreadline-dev zlib1g-dev bison flex libssl-dev openssl
```


_TDE doesn’t exist for Postgresql in its original package_. It has to be downloaded and used. And its’ pre-built configuration is only available for **`server-side encryption`**, while the server runs. There are other configurations of a TDE as well, such as _client side_, _tablespace_ etc.

To begin with, you first need to install Postgresql TDE from a third-party tool known as **`CyberTec`**



INSTALLING POSTGRESQL 12.X TDE FROM SOURCE

**`PostgreSQL 12.x TDE`** is a _version of PostgreSQL_ which _supports transparent data encryption_. In many practical business cases it is necessary to **`encrypt data on disk`**. PostgreSQL TDE has been designed to do exactly that in the most efficient way possible.


### 1. Download the source code
To download PostgreSQL TDE, we can go to our ​[Transparent Data Encryption Installation Guide website](https://www.cybertec-postgresql.com/en/transparent-data-encryption-installation-guide/) and download the latest tar file. If you prefer to use command line, you can also simply use **`wget`** to get the file.

```
wget https://download.cybertec-postgresql.com/postgresql-12.3_TDE_1.0.tar.gz
```

### 2. Extracting and configuring PostgreSQL 12 TDE

Once you have downloaded the file, you can easily unpack it. Here is how it works:
```
sudo su
usermod -aG sudo postgres
```
```
sudo apt install make
sudo su - postgres
tar xvfz postgresql-12.3_TDE_1.0.tar.gz
cd postgresql-12.3_TDE_1.0/
```
##### For Linux:

On Linux, compiling PostgreSQL is pretty straight forward. Make sure that all the dependencies outlined in the documentation are met. A directory will be created containing the entire source code of PostgreSQL TDE. Enter
the directory and execute the following command:

```
sudo apt-get update -y
sudo apt-get install libldap2-dev libperl-dev
sudo apt-get install python3-dev
```

```
# in pgsql directory all files will be installed
./configure --prefix=/usr/local/pgsql --with-openssl --with-perl \
   --with-python --with-ldap
```

> **_NOTE:_**  If something goes wrong during configuring, it is very likely that you have missed a package. 

### 3. Compiling the code

Once the “configure” step has been executed successfully, you can easily compile the code:

```
sudo make install
```
This might take a while to complete. After the make command is complete, you will need to switch to the `contrib directory` within the extracted package and issue the make command again as follows

```
cd contrib
sudo make install
```

If you want to use more than one CPU core, you can add the -j flag to “make” to compile code in parallel. 

**`PostgreSQL is now ready`**, and you can already _prepare your server infrastructure_ by adjusting the `$PATH` environment variables so you can easily _access the database_

```
ls -lh /usr/local/pgsql/
```
> **_NOTE:_** If you see (`bin` `include` `lib` `share`) means we have installed the customized postgresql


### 4. Setting up key management


After the completion of this command, you now need to set up the key that will be used for encryption. This step is pretty simple since all you have to do is **`just write a file that can output the key value`**. To do this, you can create a file as follows. _Before creating your database instance_, you have to write some code to make sure that the _key can be read by the database during startup and instance creation_.

Here is the most simplistic example possible:

```
cat /somewhere/provide_key.sh 
```

```
#!/bin/sh 
echo 882fb7c12e80280fd664c69d2d636913e2387b34263j6543
```

All you need is a program that prints the key to stdout – and that’s it! Make sure that _PostgreSQL is able to execute this program_
```
chmod +x /somewhere/provide_key.sh
```

> **_NOTE:_**: You don’t have to write a shell script – you can use any kind of executable such as a C, Go or Python.


### 5. Creating a database instance / cluster


Now log in as postgres user:
```
sudo su - postgres
```
After this step, you now need to initialize your database in a **`directory of our choice`**. This is essential to be on the _safe side_, that is to _reduce any permission errors_ during the running of the postgres server. 
Create the directory and add the attributes to it for access later on:

```
sudo mkdir /usr/local/postgres
sudo chmod 775 /usr/local/postgres
sudo chown postgres /usr/local/postgres
```

Now you can initialize your database with the key you created in the `provide_key.sh` file above. Before that, you also need to set the export path for the database

```
export PATH=$PATH:/usr/local/pgsql/bin
```

And finally hit:

```
initdb -D /usr/local/postgres -K /provide_key.sh
```

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
pg_ctl -D /usr/local/postgres -l logfile start
```
