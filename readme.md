# Store.londonparkour.com

This repository contains the infrastructure and code to fully deploy the store.londonparkour.com website.

## docker-compose
---
Use this docker compose file to start up the container infrastructure.

### Starting the containers

```bash
docker compose up -d
```

### Switching from DEV to LIVE

The `.env` file has a `COMPOSE_PROFILES=dev` variable. Switch to `COMPOSE_PROFILES=live`


### Certbot
---
The letsencrypt certbot needs to have the nginx port 80 running first on the live server. 

Rename the `config/nginx/nginx-conf-live/nginx.conf.80` file to `config/nginx/nginx-conf-live/nginx.conf` and start the certbot container.

```bash
docker compose up certbot -d
``` 

Check the logs to make sure the certificate has been obtained:

```bash
docker logs certbot
```

Once the certificate has been obtained, switch the `config/nginx/nginx-conf-live/nginx.conf.443` to being the `config/nginx/nginx-conf-live/nginx.conf`.

Restart the webserver container.

```bash
docker compose restart webserver_live
```

You should now have a certificate running on the server.

## PHP
---
You can configure the `php.ini` file by editing the `/config/php/uploads.ini` file and restarting the `wordpress` container.

## Backups / Recover

You can use the [vump](https://github.com/IORoot/docker-vump) tool to backup and recover the website.

It will take a copy of the database and the data files, put them into a new container and push that container up to the container registry. 
The specific settings are all held in the `.env` file on the server.

## .env

The `.env` file contains all of the main website variables used for both running the website and backing up.

The file should contain the following:

```bash
# ╭──────────────────────────────────────────────────────────╮
# │                      DOCKER COMPOSE                      │
# ╰──────────────────────────────────────────────────────────╯

    # Profile to use
    # Switch between 'dev' and 'live' which will activate
    # the correct webserver and certbot if on live.
    # COMPOSE_PROFILES=live
    COMPOSE_PROFILES=dev
    
    # MacOS M1/M2 chips need the linux/arm64/v8 specifically
    # Linux will use the default linux/amd64 versions.
    # PLATFORM=linux/amd64
    PLATFORM=linux/arm64/v8
    
    # Sets database name so we do not clash with other
    # website databases on the same mysql instance.
    DATABASE_NAME=wordpress

    # DOCKER MYSQL VARIABLES
    # Use these on both the dev and live environments
    # to make sure they work seemlessly.
    MYSQL_ROOT_PASSWORD=????????
    MYSQL_USER=????????
    MYSQL_PASSWORD=????????

    # DOCKER WORDPRESS VARIABLES
    # Change these from localhost to the website domain
    # so that wordpress doesn't use hard-coded links.
    WP_SITEURL=http://localhost
    WP_HOME=http://localhost

    # DOCKER CERTBOT
    # This is only used on live. The live version will
    # use flag=force-renewal and the site domain. 
    CERTBOT_EMAIL=me@gmail.com
    CERTBOT_DOMAIN=localhost
    CERTBOT_FLAG=staging


# ╭──────────────────────────────────────────────────────────╮
# │                       VUMP BACKUPS                       │
# ╰──────────────────────────────────────────────────────────╯

    # Name of the VOLUME that stores all of the website data. 
    # This will be backed up to the registry as an image.
    VUMP_WEBSITE_VOLUME_NAME="website-data"

    # The name of the CONTAINER of the database. MySQLDump will be used 
    # on the DB to backup and put into the registry image.
    VUMP_DATABASE_CONTAINER_NAME="mariadb"

    # To get the MySQLDump to work we need the database username
    # credentials for access.
    VUMP_DB_USERNAME="????????"

    # To get the MySQLDump to work we need the database password
    # credentials for access.
    VUMP_DB_PASSWORD="????????"

    # This is the name of the database that will be dumped into the
    # backup file. This will be the wordpress website database.
    VUMP_DB_DATABASE="wordpress"

    # This is the name of the registry to send the backups to
    # Can be a full URL, but default is dockerhub.
    VUMP_REGISTRY_REPO="dockerhub_username/backup"

    # The tag to add to the container image when pushing up to the
    # container registry repository on dockerhub.
    VUMP_PULL_TAG="latest"
```