### Download
```sh
$ sudo yum install postgresql-server
```
### Enable service
```sh
$ sudo systemctl enable postgresql.service
```
### InitDB
```sh
$ sudo /usr/bin/postgresql-setup initdb
```
### Start service
```sh
$ sudo systemctl start postgresql.service
```
### Configuration
```sh
$ sudo vi /var/lib/pgsql/data/postgresql.conf
```
#### Connection Settings
find & replace

As-is:

`listen_addresses = 'localhost'`

To-be:

`listen_addresses = '*'`
### Change password
```sh
$ sudo vi /var/lib/pgsql/data/pg_hba.conf
```
find & replace

As-is:

```
local   all             all                                     peer
```

To-be:

```
local   all             all                                     trust
```
```sh
$ sudo systemctl restart postgresql.service
$ psql -U postgres

postgres=# ALTER USER postgres with password 'yourPasswd';
postgres=# \q
```
```sh
$ sudo vi /var/lib/pgsql/data/pg_hba.conf
```
find & replace

As-is:

```
local   all             all                                     trust
```

To-be:

```
local   all             all                                     md5
```
and, add row
```
host    all             all             0.0.0.0/0               md5
```
```sh
$ sudo systemctl restart postgresql.service
```
```sh
$ psql -U postgres
Password for user postgres:  # <== new password
psql (9.2.18)
Type "help" for help.

postgres=#
```
### CREATE DATABASE
```sh
postgres=# CREATE DATABASE test1 WITH OWNER = postgres ENCODING = 'UTF8';
CREATE DATABASE
postgres=# 
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-------------+----------+----------+-------------+-------------+-----------------------
 test1        | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
 template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
(4 rows)
```
### Use Database
```sh
postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# \q
```
### Client
`https://www.pgadmin.org/download/`
