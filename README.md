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

You need to define four variables: a password with which to encrypt the
backups before uploading to s3, a list of databases to backup, a user to use
to backup the databases, and the name of an S3 bucket where the backups
will be saved.

```shell
# Choose a long, strong password to encrypt the backups.
PASSPHRASE="SomeReallyGoodPassword"

# List of databases to backup:
DBS=('test_database')

# User with access permissions on the databases.
BACKUP_USER="postgres"

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

## Usage

Running a backup is easy! After configuring everything just run the
`pgbackup` script!

Note that after decrypting and decompressing a backup you can restore it like
so:

```shell
$ psql -f database.sql
```

### Crontab
You might also find it helpful to run the backup as a cron job. Below is an
example that runs the backup everyday at 0245, but you could obviously adjust
it to suit your needs. I also have my s3 bucket set to delete backups older
than three months (you could also have them transition to Glacier-class
storage). Assuming you have the script deployed to your home directory:

```
45 2 * * * cd /home/user/postgresql-backup && /home/user/postgresql-backup/pgbackup
