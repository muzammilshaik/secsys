---
author: muju
title: "psql"
description: ""
categories: ["Cheatsheet"]
image:
  path: /assets/sam/cslogo.webp
  lqip: data:image/webp;base64,UklGRrgAAABXRUJQVlA4IKwAAACwBQCdASoUABQAPpE8mEiloyIhKA1QsBIJQBOmUI9LDMxygR2TUG+/ASwZeFfY+ZZxYNEbCgAAudBPH7h7qkaH0PDpyyq6RhcdUxV3fuoYNyS2UMvGHHRpYpcuMobdbrEI2XA9pZTO7JWBbQC+lek8NkHzNXHak0HJURnPX+fehuSaqziRVP3OZAqmtQRzRTek7lDJeGJwAKHPllHlqR55w1XwQ/ZvvcwKXQA
tags: ["cheatsheet"]
published: true
hidden: true
toc: true  #table of content
---

# Postgres Cheatsheet

This is a collection of the most common commands I run while administering Postgres databases. The variables shown between the open and closed tags, "<" and ">", should be replaced with a name you choose. Postgres has multiple shortcut functions, starting with a forward slash, "\". Any SQL command that is not a shortcut, must end with a semicolon, ";". You can use the keyboard UP and DOWN keys to scroll the history of previous commands you've run.


## Setup

##### installation, [Ubuntu](http://www.postgresql.org/download/linux/ubuntu/){:target="_blank"} [Community](https://help.ubuntu.com/community/PostgreSQL){:target="_blank"}
``` shell
sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ wily-pgdg main" > \
  /etc/apt/sources.list.d/pgdg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y postgresql-9.5 postgresql-client-9.5 postgresql-contrib-9.5

sudo su - postgres
psql
```

##### [connect](http://www.postgresql.org/docs/current/static/app-psql.html){:target="_blank"}
```sql
psql

psql -U <username> -d <database> -h <hostname>

psql --username=<username> --dbname=<database> --host=<hostname>
```

##### disconnect
```sql
\q
\!
```

##### clear the screen
```sql
(CTRL + L)
```

##### info
```sql
\conninfo
```

##### [configure](http://www.postgresql.org/docs/current/static/runtime-config.html){:target="_blank"}

```shell
sudo nano $(locate -l 1 main/postgresql.conf)
sudo service postgresql restart
```

##### debug logs
```shell
# print the last 24 lines of the debug log
sudo tail -24 $(find /var/log/postgresql -name 'postgresql-*-main.log')
```

## Recon

##### show version
```
SHOW SERVER_VERSION;
```

##### show system status
```sql
\conninfo
```

##### show environmental variables
```sql
SHOW ALL;
```

##### list users
```sql
SELECT rolname FROM pg_roles;
```

##### show current user
```sql
SELECT current_user;
```

##### show current user's permissions
```
\du
```

##### list databases
```sql
\l
```

##### show current database
```sql
SELECT current_database();
```

##### show all tables in database
```sql
\dt
```

##### list functions
```sql
\df <schema>
```


## Databases

##### list databasees
```sql
\l
```

##### connect to database
```sql
\c <database_name>
```

##### show current database
```sql
SELECT current_database();
```

##### [create database](http://www.postgresql.org/docs/current/static/sql-createdatabase.html){:target="_blank"}
```sql
CREATE DATABASE <database_name> WITH OWNER <username>;
```
##### [delete database](http://www.postgresql.org/docs/current/static/sql-dropdatabase.html){:target="_blank"}
```sql
DROP DATABASE IF EXISTS <database_name>;
```

##### [rename database](http://www.postgresql.org/docs/current/static/sql-alterdatabase.html){:target="_blank"}
```sql
ALTER DATABASE <old_name> RENAME TO <new_name>;
```

## Users

##### list roles
```sql
SELECT rolname FROM pg_roles;
```

##### [create user](http://www.postgresql.org/docs/current/static/sql-createuser.html){:target="_blank"}
```sql
CREATE USER <user_name> WITH PASSWORD '<password>';
```

##### [drop user](http://www.postgresql.org/docs/current/static/sql-dropuser.html){:target="_blank"}
```sql
DROP USER IF EXISTS <user_name>;
```

##### [alter user password](http://www.postgresql.org/docs/current/static/sql-alterrole.html){:target="_blank"}
```sql
ALTER ROLE <user_name> WITH PASSWORD '<password>';
```

## Permissions

##### become the postgres user, if you have permission errors
```shell
sudo su - postgres
psql
```

##### grant all permissions on [database](http://www.postgresql.org/docs/current/static/sql-grant.html){:target="_blank"}
```sql
GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <user_name>;
```

##### grant connection permissions on database
```sql
GRANT CONNECT ON DATABASE <db_name> TO <user_name>;
```

##### grant permissions on schema
```sql
GRANT USAGE ON SCHEMA public TO <user_name>;
```

##### grant permissions to functions
```sql
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO <user_name>;
```

##### grant permissions to select, update, insert, delete, on a all tables
```sql
GRANT SELECT, UPDATE, INSERT ON ALL TABLES IN SCHEMA public TO <user_name>;
```

##### grant permissions, on a table
```sql
GRANT SELECT, UPDATE, INSERT ON <table_name> TO <user_name>;
```

##### grant permissions, to select, on a table
```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO <user_name>;
```



## Schema

#####  list schemas
```sql
\dn

SELECT schema_name FROM information_schema.schemata;

SELECT nspname FROM pg_catalog.pg_namespace;
```

#####  [create schema](http://www.postgresql.org/docs/current/static/sql-createschema.html){:target="_blank"}
```sql
CREATE SCHEMA IF NOT EXISTS <schema_name>;
```

#####  [drop schema](http://www.postgresql.org/docs/current/static/sql-dropschema.html){:target="_blank"}
```sql
DROP SCHEMA IF EXISTS <schema_name> CASCADE;
```





## Tables

##### list tables, in current db
```sql
\dt

SELECT table_schema,table_name FROM information_schema.tables ORDER BY table_schema,table_name;
```

##### list tables, globally
```sql
\dt *.*.

SELECT * FROM pg_catalog.pg_tables
```

##### list table schema
```sql
\d <table_name>
\d+ <table_name>

SELECT column_name, data_type, character_maximum_length
FROM INFORMATION_SCHEMA.COLUMNS
WHERE table_name = '<table_name>';
```

##### [create table](http://www.postgresql.org/docs/current/static/sql-createtable.html){:target="_blank"}
```sql
CREATE TABLE <table_name>(
  <column_name> <column_type>,
  <column_name> <column_type>
);
```

##### create table, with an auto-incrementing primary key
```sql
CREATE TABLE <table_name> (
  <column_name> SERIAL PRIMARY KEY
);
```

##### [delete table](http://www.postgresql.org/docs/current/static/sql-droptable.html){:target="_blank"}
```sql
DROP TABLE IF EXISTS <table_name> CASCADE;
```

## Columns

##### [add column](http://www.postgresql.org/docs/current/static/sql-altertable.html){:target="_blank"}
```sql
ALTER TABLE <table_name> IF EXISTS
ADD <column_name> <data_type> [<constraints>];
```

##### update column
```sql
ALTER TABLE <table_name> IF EXISTS
ALTER <column_name> TYPE <data_type> [<constraints>];
```

##### delete column
```sql
ALTER TABLE <table_name> IF EXISTS
DROP <column_name>;
```

##### update column to be an auto-incrementing primary key
```sql
ALTER TABLE <table_name>
ADD COLUMN <column_name> SERIAL PRIMARY KEY;
```

##### insert into a table, with an auto-incrementing primary key
```sql
INSERT INTO <table_name>
VALUES (DEFAULT, <value1>);


INSERT INTO <table_name> (<column1_name>,<column2_name>)
VALUES ( <value1>,<value2> );
```
<br/><br/><br/>





## Data

##### [read all data](http://www.postgresql.org/docs/current/static/sql-select.html){:target="_blank"}
```sql
SELECT * FROM <table_name>;
```

##### read one row of data
```sql
SELECT * FROM <table_name> LIMIT 1;
```

##### search for data
```sql
SELECT * FROM <table_name> WHERE <column_name> = <value>;
```

##### [insert data](http://www.postgresql.org/docs/current/static/sql-insert.html){:target="_blank"}
```sql
INSERT INTO <table_name> VALUES( <value_1>, <value_2> );
```

##### [edit data](http://www.postgresql.org/docs/current/static/sql-update.html){:target="_blank"}
```sql
UPDATE <table_name>
SET <column_1> = <value_1>, <column_2> = <value_2>
WHERE <column_1> = <value>;
```

##### [delete all data](http://www.postgresql.org/docs/current/static/sql-delete.html){:target="_blank"}
```sql
DELETE FROM <table_name>;
```

##### delete specific data
```sql
DELETE FROM <table_name>
WHERE <column_name> = <value>;
```





## Scripting

##### [run local script, on remote host](http://www.postgresql.org/docs/current/static/app-psql.html){:target="_blank"}
```shell
psql -U <username> -d <database> -h <host> -f <local_file>

psql --username=<username> --dbname=<database> --host=<host> --file=<local_file>
```

##### backup database data, [everything](http://www.postgresql.org/docs/current/static/app-pgdump.html){:target="_blank"}
```shell
pg_dump <database_name>

pg_dump <database_name>
```

##### backup database, only data
```shell
pg_dump -a <database_name>

pg_dump --data-only <database_name>
```

##### backup database, only schema
```shell
pg_dump -s <database_name>

pg_dump --schema-only <database_name>
```

##### [restore database data](http://www.postgresql.org/docs/current/static/app-pgrestore.html){:target="_blank"}
```shell
pg_restore -d <database_name> -a <file_pathway>

pg_restore --dbname=<database_name> --data-only <file_pathway>
```

##### restore database schema
```shell
pg_restore -d <database_name> -s <file_pathway>

pg_restore --dbname=<database_name> --schema-only <file_pathway>
```

##### [export table into CSV file](http://www.postgresql.org/docs/current/static/sql-copy.html){:target="_blank"}
```sql
\copy <table_name> TO '<file_path>' CSV
```

##### export table, only specific columns, to CSV file
```sql
\copy <table_name>(<column_1>,<column_1>,<column_1>) TO '<file_path>' CSV
```

##### [import CSV file into table](http://www.postgresql.org/docs/current/static/sql-copy.html){:target="_blank"}
```sql
\copy <table_name> FROM '<file_path>' CSV
```

##### import CSV file into table, only specific columns
```sql
\copy <table_name>(<column_1>,<column_1>,<column_1>) FROM '<file_path>' CSV
```


## Debugging

[Using EXPLAIN](http://www.postgresql.org/docs/current/static/using-explain.html){:target="_blank"}
[Error Reporting and Logging](http://www.postgresql.org/docs/current/static/runtime-config-logging.html){:target="_blank"}


