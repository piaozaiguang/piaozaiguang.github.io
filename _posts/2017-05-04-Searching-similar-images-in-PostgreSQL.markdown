_Suppose you have installed postgre_
### Install imgsmlr
```sh
$ sudo yum install -y gd-devel
$ cd ~
$ git clone https://github.com/postgrespro/imgsmlr
$ cd imgsmlr/
$ sudo make USE_PGXS=1 install
$ psql -U postgres
Password for user postgres: 
psql (9.2.18)
Type "help" for help.

postgres=# create extension imgsmlr;
CREATE EXTENSION
postgres=# \c test1
test1=# 
```
### Import images
Copy all image file to `data_directory`.(Check `data_directory` use command `SHOW  data_directory;`)
```sh
$ psql -U postgres
Password for user postgres: 
psql (9.2.18)
Type "help" for help.

postgres=# \c test1
You are now connected to database "test1" as user "postgres".
test1=# create table image (id serial, data bytea);
test1=# insert into image(data) select pg_read_binary_file('/var/lib/pgsql/data/test1/999.jpg');
INSERT 0 1
```
Or, use script file
```sh
$ psql -f bulk_script.sql -U postgres
Password for user postgres: 
```
### CREATE EXTENSION
```sh
test1=# CREATE EXTENSION imgsmlr;
CREATE EXTENSION
```
### Create pattern table
```sql
CREATE TABLE pat AS (
     SELECT
         id,
         shuffle_pattern(pattern) AS pattern, 
         pattern2signature(pattern) AS signature 
     FROM (
         SELECT 
             id, 
             jpeg2pattern(data) AS pattern 
         FROM 
             image
     ) x 
 );
```
### Add PK & index
```sql
ALTER TABLE pat ADD PRIMARY KEY (id);
CREATE INDEX pat_signature_idx ON pat USING gist (signature);
```
### Test
```sql
SELECT
	id,
	smlr
FROM
(
	SELECT
		id,
		pattern <-> (SELECT pattern FROM pat WHERE id = 1) AS smlr
	FROM pat
	WHERE id <> 1
	ORDER BY
		signature <-> (SELECT signature FROM pat WHERE id = 1)
	LIMIT 100
) x
ORDER BY x.smlr ASC 
LIMIT 10
```
### Additional
#### Get image use http
```sh
$ cd ~
$ git clone https://github.com/pramsey/pgsql-http
$ cd pgsql-http
$ sudo make install
$ psql -U postgres
Password for user postgres: 
psql (9.2.18)
Type "help" for help.

postgres=# create extension http;
CREATE EXTENSION

postgres=# \c test1
test1=# 
test1=# insert into image(data) SELECT textsend(content) FROM http_get('http://liugenxian.com/imgs/liugenxian_logo.png');
```

> References

* https://wiki.postgresql.org/images/4/43/Pgcon_2013_similar_images.pdf
* https://github.com/postgrespro/imgsmlr
* https://github.com/pramsey/pgsql-http
* https://m.aliyun.com/yunqi/articles/64959
