# 🐳 MHCT DB Docker &middot; [![Build Status](https://img.shields.io/docker/cloud/build/tsitu/mhct-db-docker.svg)](https://hub.docker.com/r/tsitu/mhct-db-docker/builds)

Docker images aimed at simplifying access to MHCT's various databases. Each image contains a MySQL 8.0+ server with data from [MHCT's backups](https://backups.mhct.win/).

Based on [Bavo's repo](https://github.com/bavovanachte/jacks-tools-docker).

## Images / Tags

### Standard Images (Pre-initialized)
These images have the database fully initialized during the build process. They are larger but start immediately:
```bash
- tsitu/mhct-db-docker:latest       # NIGHTLY 'mhhunthelper' dump (pre-initialized)
- tsitu/mhct-db-docker:weekly       # WEEKLY  'mhhunthelper' dump (pre-initialized)
- tsitu/mhct-db-docker:converter    # WEEKLY  'mhconverter'  dump (pre-initialized)
- tsitu/mhct-db-docker:mapspotter   # WEEKLY  'mhmapspotter' dump (pre-initialized)
```

### Slim Images (Initialize on First Run)
These images contain the SQL dumps but initialize on first container run. They build faster and are smaller, but **first startup will take several minutes** as the database is initialized:
```bash
- tsitu/mhct-db-docker:latest-slim      # NIGHTLY 'mhhunthelper' dump
- tsitu/mhct-db-docker:converter-slim   # WEEKLY  'mhconverter'  dump
- tsitu/mhct-db-docker:mapspotter-slim  # WEEKLY  'mhmapspotter' dump
```

## Installation

```bash
$ docker pull tsitu/mhct-db-docker:$DESIRED_TAG
$ docker run -p 3306:3306 -d tsitu/mhct-db-docker:$DESIRED_TAG
```

This should set up a server that you can query on `localhost:3306`.

**Important for `-slim` images:** The first time you start a slim image, the database will initialize from the dump file. This may take **5-15 minutes** for the maps and converter and up to **2 hours** for hunthelper depending on your system. You can monitor progress with `docker logs -f <container_name>`.

*Note:* Replace `localhost` with `docker-machine ip` if you are using Docker Toolbox (most common IP is `192.168.99.100`)

## Running Multiple Databases At Once

If you want to start all (or most) of the databases and connect to them from an external tool you can use something like:

```bash
$ docker run -p 3306:3306 -d --name mhct-mhhunthelper tsitu/mhct-db-docker:latest
$ docker run -p 3307:3306 -d --name mhct-converter tsitu/mhct-db-docker:converter
$ docker run -p 3308:3306 -d --name mhct-mapspotter tsitu/mhct-db-docker:mapspotter
```

MySQL's default port is 3306. You will need to use the first number after the `-p` (3306, 3307, or 3308) to connect to the database you want.

### With Persistent Data Volumes

By default, MySQL data is stored in a Docker volume. **If you don't mount a host directory, the data will be deleted when the container is removed.** To persist data on your host machine:

```bash
$ docker run -p 3308:3306 -d --name mhct-mapspotter -v ./data/mapspotter:/var/lib/mysql tsitu/mhct-db-docker:mapspotter-slim
```

This mounts the `./data` directory on your host to preserve data across container deletions.

## Credentials

```bash
ENV MYSQL_ROOT_PASSWORD=secret
ENV MYSQL_USER=admin
ENV MYSQL_PASSWORD=admin

ENV MYSQL_DATABASE=mhhunthelper  # for 'latest' and 'weekly' image tags
# OR
ENV MYSQL_DATABASE=mhconverter   # for 'converter' image tag
# OR
ENV MYSQL_DATABASE=mhmapspotter  # for 'mapspotter' image tag
```
