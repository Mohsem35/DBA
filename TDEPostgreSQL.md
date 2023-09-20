
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
./configure --prefix=/usr/local/pg12tde --with-openssl --with-perl \
   --with-python --with-ldap
```
 

If something goes wrong during configuring, it is very likely that you have missed a package. 
