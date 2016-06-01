# PostgreSQL Backup

Backup PostgreSQL databases to S3.

## Dependencies

### s3cmd

[s3cmd](https://github.com/s3tools/s3cmd) is used to actually upload the
backups to AWS, so you need to install and configure it first. Install it
with your favorite package manager and then configure with:

```shell
$ s3cmd --configure
```

## Configuration

Configuration for the database backup is saved in your home directory in the
`.pgbackup` file. It is basically just another bash script that defines the
variables that we need which we source before running the backup.

You need to define three variables: a password with which to encrypt the
backups before uploading to s3, a list of databases to backup and the name of
an S3 bucket where the backups will be saved.

```shell
# Choose a long, strong password to encrypt the backups.
PASSPHRASE="SomeReallyGoodPassword"

# List of databases to backup:
DBS=('test_database')

# You can list your buckets with `s3cmd ls`.
BUCKET="your-backup-bucket"
```

### `pg_dump`

You also need to make sure that the username and password for the `pg_dump`
user is configured, probably through the use of a
[`.pgpass` file](https://www.postgresql.org/docs/current/static/libpq-pgpass.html).
Don't forget that this file needs to have `600` permissions or it will fail.
The script does no checking of the validity of the pgpass but if the values
are not set then `pg_dump` will obviously fail.
