---
categories:
    - database
date: 2024-03-13T15:47:20Z
description: Use mysqldump to export data from MySQL/MariaDB, and make it compatible with SQLite
keywords: mysqldump, export, sqlite, mysql, mariadb
tags:
    - database
    - mysql
    - mariadb
    - sqlite
title: Export Data from MySQL by mysqldump
---



# how to export data from the mysql

## use mysqldump

Just for the mysql/mariadb using:

```bash
docker container run -it --rm mysql:8 bash -c "echo -e '[mysqldump]\nuser=aurora\npassword=aurora' > ~/.my.cnf && \
    mysqldump -h 192.168.91.199 your_database_name" > exported.sql
```

For the SQLite compatibility:

```bash
docker container run -it --rm mysql:8 bash -c "echo -e '[mysqldump]\nuser=aurora\npassword=aurora' > ~/.my.cnf && \
    mysqldump --compact --no-create-info --skip-add-locks --complete-insert -h 192.168.91.199 your_database_name \
        table1 \
        table2 \
        table3 \
        table4" > exported.sql

# Some Compatible Options for SQLite
## because the sqlite cannot parse the "\'", we should use double single quote instead
sed -i "s/\\\'/''/g" exported.sql
```

## Explanation in Details

`docker container run -it --rm mysql:8 bash -c`:
- `mysql:8` is the image name and tag
- `-it` is to run the container in interactive mode
- `--rm` is to remove the container after it is stopped
- `bash -c` is to run the command in the container

The left is a combined command to create a `.my.cnf` file and run `mysqldump` command.
- `echo -e '[mysqldump]\nuser=aurora\npassword=aurora' > ~/.my.cnf` is to create a `.my.cnf` file at `$HOME` with the content
    ```
    [mysqldump]
    user=aurora
    password=aurora
    ```
- `--compact` is to use the compact format, which aims to remove the Comment Syntax, the `/*!` and `*/` are removed
- `--no-create-info` is to skip the `CREATE TABLE` statement, because the statement is not compatible with SQLite
- `--skip-add-locks` is to skip the `LOCK TABLES` statement, because the statement is not compatible with SQLite
- `--complete-insert` is to use the `INSERT INTO` statement with the column names
- `-h` is to specify the db host
- `your_database_name` is the database name
- `table1 table2 table3 table4` are the exported table names, separated by space; if not specified, all tables will be exported
- `> exported.sql` is to redirect the output to a file named `exported.sql`, at the host current directory
