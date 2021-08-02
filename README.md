![Docker Pulls](https://img.shields.io/docker/pulls/acouvreur/minio-backup-local)

# minio-backup-local

Backup Minio buckets to the local filesystem with periodic rotating backups, based on [docker-postgres-backup-local](https://github.com/prodrigestivill/docker-postgres-backup-local).
Backup multiple buckets from the same host by setting the bucket names in `MINIO_BUCKET` separated by commas or spaces.

Supports the following Docker architectures: `linux/amd64`, `linux/arm64`, `linux/arm/v7`, `linux/s390x`, `linux/ppc64le`.

## Usage

Docker:

```sh
docker run -e MINIO_BUCKET=default -e MINIO_DIR=/minio_data -v minio_data:/minio_data acouvreur/minio-backup-local:latest
```

Docker Compose:

```yaml
version: '3.7'
services:
    s3:
      image: minio/minio
      volumes:
        - minio_data:/data
      command: server /data --console-address ":9001"
      ports: 
        - 9000:9000
        - 9001:9001

    minio-backup:
      image: acouvreur/minio-backup-local:latest
      restart: always
      volumes:
        - minio_data:/minio_data
        - minio_backups:/backups
      environment:
        - MINIO_DIR=/minio_data
        - MINIO_BUCKET=default test second

volumes:
  minio_data:
  minio_backups:
```

### Environment Variables

| env variable | description |
|--|--|
| BACKUP_DIR | Directory to save the backup at. Defaults to `/backups`. |
| BACKUP_SUFFIX | Filename suffix to save the backup. Defaults to `.sql.gz`. |
| BACKUP_KEEP_DAYS | Number of daily backups to keep before removal. Defaults to `7`. |
| BACKUP_KEEP_WEEKS | Number of weekly backups to keep before removal. Defaults to `4`. |
| BACKUP_KEEP_MONTHS | Number of monthly backups to keep before removal. Defaults to `6`. |
| HEALTHCHECK_PORT | Port listening for cron-schedule health check. Defaults to `8080`. |
| MINIO_DIR | Minio buckets directory path. Required. |
| MINIO_BUCKET | Backup multiple buckets from the same host by setting the bucket names in `MINIO_BUCKET` separated by commas or spaces. |
| SCHEDULE | [Cron-schedule](http://godoc.org/github.com/robfig/cron#hdr-Predefined_schedules) specifying the interval between backups. Defaults to `@daily`. |
| TZ | [POSIX TZ variable](https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html) specifying the timezone used to evaluate SCHEDULE cron (example "Europe/Paris"). |

### Manual Backups

By default this container makes daily backups, but you can start a manual backup by running `/backup.sh`.

This script as example creates one backup as the running user and saves it the working folder.

```sh
docker run --rm -v "$PWD:/backups" -u "$(id -u):$(id -g)" -e MINIO_BUCKET=default -e MINIO_DIR=/minio_data -v minio_data:/minio_data  acouvreur/minio-backup-local /backup.sh
```

### Automatic Periodic Backups

You can change the `SCHEDULE` environment variable in `-e SCHEDULE="@daily"` to alter the default frequency. Default is `daily`.

More information about the scheduling can be found [here](http://godoc.org/github.com/robfig/cron#hdr-Predefined_schedules).

Folders `daily`, `weekly` and `monthly` are created and populated using hard links to save disk space.