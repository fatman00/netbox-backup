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

## test it out
Get this backup example up and runnig on killercoda.com
```
sudo apt update
sudo apt upgrade -y

# Install latest version of docker-compose
pip3 install --upgrade --force-reinstall --no-cache-dir docker-compose && ln -sf /usr/local/bin/docker-compose /usr/bin/docker-compose

# Download backup from github
git clone https://github.com/fatman00/netbox-backup.git

# Download netbox docker repo
git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
# Rename override example file.
mv docker-compose.override.yml.example docker-compose.override.yml

# Try older version
export VERSION=v3.3.10

# Pull docker containers
docker-compose pull

# Restore SQL data
docker-compose up -d postgres
docker-compose exec -T postgres sh -c 'pg_restore -v -Fc -cU $POSTGRES_USER -d $POSTGRES_DB' < backup/sqldump.pgdump

#Restore media files
docker-compose up -d
docker-compose exec -T netbox tar x -zvf - -C /opt/netbox/netbox/media < backup/media.tar.gz

#Create new super user
docker-compose exec netbox python manage.py createsuperuser 
```
