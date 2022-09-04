# mysql dump with a schedule

A docker image to dump mysql database on a periodic.

## Usage

There are two methods to run this container.

**Examples**

* **Dump a remote  database:**

```shell
  $ docker run -d \
    -e DUMP_DEBUG="true" \
    -e MYSQL_HOST="x.x.x.x" \
    -e MYSQL_PORT="xxx"  \
    -e MYSQL_USER="myuser" \
    -e MYSQL_PASSWORD="mypassword" \
    -e MYSQL_DATABASE="dbname" \
    -e OPTIONS="<mysqldump_options>" \
    -e SCHEDULE="dump_schdule" \
    -e RESERVES="dump_file_reserve_number" \
    -v /your/backup-folder:/backups betacz/mysql-dump:latest
```
* **Dump a database that's running on a container:**

```shell
  $ docker run -d \
    --link your-mysql-db:db
    -e DUMP_DEBUG="true" \
    -e MYSQL_USER="myuser" \
    -e MYSQL_PORT="xxx"  \
    -e MYSQL_PASSWORD="mypassword" \
    -e MYSQL_DATABASE="dbname" \
    -e OPTIONS="mysqldump_options" \
    -e SCHEDULE="dump_schdule" \
    -e RESERVES="dump_file_reserve_number" \
    -v /your/backup-folder:/backups betacz/mysql-dump:latest
```

The dumped file will be named as `"db_backup_<timestamp>.gz"`.

**Additional,** you can do dump immediately like this:

```shell
  $ docker run [options] betacz/mysql-dump:latest dump
```

The `[options]` is as same as above.


## Environments

Some environment variable has default value, so you needn't set all of them in most cases.

* `MYSQL_HOST`: Database's hostname or address. If you want to backup a database in container at same host, use `"--link your-db:db"` instead of.
* `MYSQL_PORT`: default is 3306.
* `MYSQL_USER`: Database's username, default is `"root"`.
* `MYSQL_PASSWORD`: Database's password. If ignore, then use `MYSQL_PASSWORD_FILE`.
* `MYSQL_PASSWORD_FILE`: Docker swarm secret name.
* `MYSQL_DATABASE`: Database's name, default is `"--all-databases"` that means all database. you can use `"db_mame"` to dump only one database or `"--databases db1 db2..."` to dump multiple databases.
* `OPTIONS`: Any mysqldump's options you want to use, no default.
* `SCHEDULE`: Explained as shown below. default is `"daily"`.
* `RESERVES`: Dump file reserve numbers. default is `7`.
* `DUMP_DEBUG`: If set `"true"` then show debug info. default is `"false"`.
* `CHECK_URL`: [healthchecks.io](https://healthchecks.io) url or similar cron monitoring to perform a GET after a successful sync.
* `TZ`: set the timezone to use for the cron and log `America/Argentina/Buenos_Aires`.

### Schedule syntax:

* `"hourly"`: 0 minute every hour.
* `"daily"`: 02:00 every day.
* `"weekly"`: 03:00 on Sunday every week.
* `"monthly"`: 05:00 on 1st every month.
* `"0 5 * * 6"`: crontab syntax.

## Encrypt

If you dump a database that's include secret data, maybe you want to encrypt it.

There is a envrionment variable `"ENCRYPT_KEY"` can do this as shown below:

```shell
   $ docker run -e ENCRYPT_KEY=your-key [other-options] betacz/mysql-dump:latest
```

The dumped file will be encrypted with `AES-256-CBC` and named with a `'.aes'` extension.

Decrypt file is also very simple, see example below:

```shell
   $ docker run -e ENCRYPT_KEY=your-key \
     -v /your/backup-folder:/backups \
     -v /your/output:/output
     betacz/mysql-dump:latest \
     decrypt db_backup_<timestamp>.gz.aes /output
```

The `"decrypt"` accepts one or two arguments: `decrypt <source file> [dest-folder]`. If you don't assign `dest-folder`, the decrypted file will be saved in same folder with source file.

## License
Released under the MIT License.

Copyright Huang lijun https://github.com/hlj
