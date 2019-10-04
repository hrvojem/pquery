# Pquery-Dynamic

## How to checkout the code ?

The code is on github and can be cloned using below link.

```bash
git clone https://github.com/rahulmalik87/pquery.git pquery_d
```

## How to compile the code?

It is important to understand that pquery-dynamic supports features on both
5.7 and 8.0 versions. Hence, before starting the runs we need to compile
pquery-dynamic with the correct mysqld build.

To compile on 5.7:

```bash
cd pquery_d
cmake -DBASEDIR=<path_to_5.7_build_basedir> -DPERCONASERVER=ON .
make -j
```

To compile on 8.0:

```bash
cmake -DBASEDIR=<path_to_8.0_build_basedir> -DPERCONASERVER=ON .
make -j
```

Example:

```bash
cmake -DBASEDIR=/home/mohit.joshi/PS280819-percona-server-8.0.16-7-linux-x86_64-debug/ -DPERCONASERVER=ON .
make -j
```

## How to start the runs?

### Pre-requisites:

Before starting the runs on pquery-dynamic, make sure you have started the MySQL server. The runs are by default executed on "test" DB, however due to existing pquery framework limitations, pquery-dynamic is unable to create "test" DB automatically. As a workaround, create "test" DB manually.

This is a temporary work-around until we figure out a way to automate it.

### Starting pquery-dynamic:

#### Basic run:

```bash
cd pquery_d/src
./pquery2-ps --tables 30 --logdir=$PWD/log --records 200 --threads 10 --seconds 200 --socket /tmp/mysql_22000.sock -k
```

* `--tables` -  This option will create the default number of tables in test DB
* `--records` - This will insert no. of rows in each table
* `--threads` - Number of parallel threads
* `--seconds` - Number of seconds the test will run on the generated load
* `--socket`  - Socket of the running mysqld instance
* `-k`        - Tells pquery-dynamic to create dynamic load. There is an option
                to create default load from a source file as well. As of now,
                this is not default behaviour and we are providing this option
                explicitly

#### Examples of some specific run:

Example #1 - In case we want to generate only temporary tables. This can be
quite helpful as this cannot be done in sysbench. The main reason why this
could not be done in sysbench is that the threads that create the tables are
different from threads that execute the DMLs.
Temporary tables are session specific.

```bash
./pquery2-ps --tables 30 --logdir=$PWD/log --records 200 --threads 10 --seconds 200 --socket /tmp/mysql_22000.sock -k --only-temp-tables
```

Example #2 - In case, we want to run only DDLs

```bash
./pquery2-ps --tables 30 --logdir=$PWD/log --records 200 --threads 10 --seconds 200 --socket /tmp/mysql_22000.sock -k --no-insert --no-update --no-delete --no-select
```

The above options will not allow SELECT,INSERT,UPDATE,DELETE queries

Example #3 - In case, we want to set a variable dynamically while the load is
running. This can be achieved with the option `--mso` (stands for MySQL Server
Option).

Syntax: `--mso=<server_variable>=<val_1>=<val_2>=<val_n>`

Assume, we would like to see the behavior of temporary tablespace encryption
being set to ON/OFF while the load is running on temp tables:

```bash
./pquery2-ps --tables 30 --logdir=$PWD/log --records 200 --threads 10 --seconds 200 --socket /tmp/mysql_22000.sock -k --only-temp-tables --mso=innodb_temp_tablespace_encrypt=ON=OFF
```

Example 4 - Example of a complex run

```bash
./pquery2-ps --tables 30 --logdir=$PWD/log --records 200 --threads 20 --seconds 300 --socket /tmp/mysql_21000.sock --seed 8 --mso=innodb_redo_log_encrypt=ON=OFF --recreate-table=50 --rotate-redo-log-key=50 --optimize=10 --rename-column=100 --add-index=100 --drop-index=100 --add-column=100 --rotate-master-key=50 --set-variable=50 --primary-key-probablity=100 --undo-tbs-sql=100 -k
```

* `--seed`           - This is a random data generator. Different values will
                       create different schema each time
* `--recreate-table` - This will execute recreate table DDL
* `--optimize`       - This will execute optimize table DDL
* `--rename-column`  - This will execute rename column DDL
* `--set-variable`   - This sets the frequency of changing the server variable
                       by using the `--mso` option
* `--undo-tbs-sql`   - This will create/drop/active/inactive undo tablespaces

#### How to check the Logs?

While executing the runs, we can choose the directory where the logs must be
created using `--logdir` option. By default, the logs are created in the `src`
directory if not explicitly provided and only the DDLs are logged; however if
we wish to log every SQL statement, pass the option `-A` (stands for All).
The SQLs are written in the log file `default.node.tld_general.log`.

Apart from general log, it also has thread specific logs which contains the
SQLs executed by each thread. For example - `default.node.tld_thread-0.sql`.
