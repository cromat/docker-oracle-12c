Oracle Standard Edition 12c Release 2
============================

This fork is made due to the errors that I encountered while running docker from MaksymBilenko/docker-oracle-12c

The following errors occured while I was trying to run docker image:

ORA-03114: not connected to ORACLE
...
3% complete 
DBCA Operation failed.

and of course, if you didn't check the logs, you will get following error when trying to connect with sqlplus:

Unable to start listener TNS Adapter error

I got it working by changing dbca command in entrypoint.sh

### Installation

Download or clone this repository, open terminal within directory and run:

    docker build -t myrepo/oracle-12c .

Run with 8080 and 1521 ports opened:

    docker run --name oracle-12c -d -p 8080:8080 -p 1521:1521 myrepo/oracle-12c
    
Creation of Oracle DB takes about 10 minutes so you will not be able to connect to it until it's complete.
You can watch creation status with following command:

    docker logs [container ip or name]

Run with data on host and reuse it:

    docker run --name oracle-12c -d -p 8080:8080 -p 1521:1521 -v /my/oracle/data:/u01/app/oracle myrepo/oracle-12c

Run with Custom DBCA_TOTAL_MEMORY (in Mb):

    docker run --name oracle-12c -d -p 8080:8080 -p 1521:1521 -v /my/oracle/data:/u01/app/oracle -e DBCA_TOTAL_MEMORY=4096 myrepo/oracle-12c

Connect database with following setting:

    hostname: localhost
    port: 1521
    sid: xe
    service name: xe
    username: system
    password: oracle

To connect using sqlplus:

<pre>
sqlplus system/oracle@//localhost:1521/xe
</pre>

Password for SYS & SYSTEM:

    oracle

Connect to Oracle Application Express web management console with following settings:

    http://localhost:8080/apex
    workspace: INTERNAL
    user: ADMIN
    password: 0Racle$

Apex upgrade up to v 5.*

    docker run -it --rm --volumes-from ${DB_CONTAINER_NAME} --link ${DB_CONTAINER_NAME}:oracle-database -e PASS=YourSYSPASS quay.io/maksymbilenko/docker-oracle-apex install
Details could be found here: https://github.com/MaksymBilenko/docker-oracle-apex

Connect to Oracle Enterprise Management console with following settings:

    http://localhost:8080/em
    user: sys
    password: oracle
    connect as sysdba: true

By Default web management console is enabled. To disable add env variable:

    docker run -d -e WEB_CONSOLE=false -p 1521:1521 -v /my/oracle/data:/u01/app/oracle myrepo/oracle-12c
    #You can Enable/Disable it on any time

Start with additional init scripts or dumps:

    docker run -d -p 1521:1521 -v /my/oracle/data:/u01/app/oracle -v /my/oracle/init/SCRIPTSorSQL:/docker-entrypoint-initdb.d myrepo/oracle-12c
    
By default Import from `docker-entrypoint-initdb.d` is enabled only if you are initializing database (1st run).

To customize dump import use `IMPDP_OPTIONS` env variable like `-e IMPDP_OPTIONS="REMAP_TABLESPACE=FOO:BAR"`
To run import at any case add `-e IMPORT_FROM_VOLUME=true`

**In case of using DMP imports dump file should be named like ${IMPORT_SCHEME_NAME}.dmp**

**User credentials for imports are  ${IMPORT_SCHEME_NAME}/${IMPORT_SCHEME_NAME}**

If you have an issue with database init like DBCA operation failed, please reffer to this [issue](https://github.com/MaksymBilenko/docker-oracle-12c/issues/16)

**TODO LIST**

* Web management console HTTPS port
* Add functionality to run custom scripts on startup, for example User creation
* Add Parameter that would setup processes amount for database (Currently by default processes=300)
* Spike with clustering support
* Spike with DB migration from 11g

**In case of any issues please post it [here](https://github.com/MaksymBilenko/docker-oracle-12c/issues).**


