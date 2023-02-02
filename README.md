# netbox-backup

Information about how to backup and restore Netbox has been described on https://github.com/netbox-community/netbox-docker/issues/262

Example backup script:
```
cd /path/to/netbox
BACKUP_PATH=backup
BACKUP_DATE=$(date +"%Y-%m-%d_%H.%M.%S")
mkdir -p "${BACKUP_PATH}"
echo "${BACKUP_DATE}"
docker-compose exec -T netbox tar c -zvf - -C /opt/netbox/netbox/media ./ > "${BACKUP_PATH}/${BACKUP_DATE}_media.tgz"
docker-compose exec -T postgres sh -c 'pg_dump -v -Fc -c -U $POSTGRES_USER $POSTGRES_DB' > "${BACKUP_PATH}/${BACKUP_DATE}.pgdump"
```
Example restore script:
```
# Restore SQL data
docker-compose up -d postgres
docker-compose exec -T postgres sh -c 'pg_restore -v -Fc -cU $POSTGRES_USER -d $POSTGRES_DB' < backup/sqldump.pgdump

# Restore media files
docker-compose up -d
docker-compose exec -T netbox tar x -zvf - -C /opt/netbox/netbox/media < backup/media.tar.gz
```