# Docker Image for Backjam

## What is Backjam?

> Backjam is an easy to use backup software, designed with security in mind. Perfect for self-hosted deployment where data  are kept in a secured and known remote location that you own.


https://backjam.com

## TL;DR

    $ curl -sSL https://raw.githubusercontent.com/getbackjam/backjam/master/docker-compose.yml > docker-compose.yml
    $ docker-compose up -d

You can find the default credentials and available configuration options in the Environment Variables section.

## Get this image

The recommended way to get the official Backjam Docker Image is to pull the prebuilt image from the Docker Hub Registry.

    $ docker pull backjam/backjam:1atest

To use a specific version, you can pull a versioned tag. You can view the list of available versions in the Docker Hub Registry.

    $ docker pull backjam/backjam:[TAG]

## How to use this image

Backjam requires access to a sqlite, MySQL or MariaDB database to store information. We'll use the [Bitnami Docker Image](https://www.github.com/bitnami/bitnami-docker-mariadb) for MariaDB for the database requirements.

### Run the application using Docker Compose

The main folder of this repository contains a functional docker-compose.yml file. Run the application using it as shown below:

    $ curl -sSL https://raw.githubusercontent.com/getbackjam/backjam/master/docker-compose.yml > docker-compose.yml
    $ docker-compose up -d

### Using the Docker Command Line

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

#### Step 1: Create a network

    $ docker network create backjam-network

#### Step 2: Create a volume for MariaDB persistence and create a MariaDB container

    $ docker volume create --name mariadb_data
    $ docker run -d --name mariadb \
      --env ALLOW_EMPTY_PASSWORD=yes \
      --env MARIADB_USER=backjam_user \
      --env MARIADB_PASSWORD=backjam \
      --env MARIADB_DATABASE=backjam_db \
      --network backjam-network \
      --volume mariadb_data:/bitnami/mariadb \
      bitnami/mariadb:latest

#### Step 3: Create volumes for Backjam persistence and launch the container

    $ docker volume create --name backjam_data
    $ docker volume create --name backjam_logs
    $ docker run -d --name backjam \
      -p 8009:8009 -p 8109:8109 \
      --env BACKJAM_DATABASE_USER=backjam_user \
      --env BACKJAM_DATABASE_PASSWORD=backjam \
      --env BACKJAM_DATABASE_NAME=backjam_db \
      --network backjam-network \
      --volume backjam_data:/server/data \
      --volume backjam_logs:/server/logs \
      backjam/backjam:latest
    
Access your application at http://your-ip:8109/
    
## Persisting your application

If you remove the container all your data will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a directory at the `/server/data` path. If the mounted directory is empty, it will be initialized on the first run. Additionally you should mount a volume for persistence of logs at the `/server/logs` path and the [MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define the Docker volumes named `mariadb_data`, `backjam_data` and `backjam_logs`. The Backjam application state will persist as long as volumes are not removed.

To avoid inadvertent removal of volumes, you can mount host directories as data volumes. Alternatively you can make use of volume plugins to host the volume data.


## Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` file present in this repository:

       mariadb:
         ...
         volumes:
    -      - mariadb_data:/bitnami/mariadb
    +      - /path/to/mariadb-persistence:/bitnami/mariadb
       ...
       backjam:
         ...
         volumes:
    -      - backjam_data:/server/data
    -      - backjam_logs:/server/logs
    +      - /path/to/backjam-data-persistence:/server/data
    +      - /path/to/backjam-logs-persistence:/server/logs
       ...
    -volumes:
    -  mariadb_data:
    -    driver: local
    -  backjam_data:
    -    driver: local
    -  backjam_logs:
    -    driver: local

### Mount host directories as data volumes using the Docker command line

#### Step 1: Create a network (if it does not exist)

    $ docker network create backjam-network

#### Step 2. Create a MariaDB container with host volume

    $ docker run -d --name mariadb \
      --env ALLOW_EMPTY_PASSWORD=yes \
      --env MARIADB_USER=backjam_user \
      --env MARIADB_PASSWORD=backjam \
      --env MARIADB_DATABASE=backjam_db \
      --network backjam-network \
      --volume /path/to/mariadb-persistence:/bitnami/mariadb \
      bitnami/mariadb:latest
    
#### Step 3. Create the Backjam container with host volumes
    
    $ docker run -d --name backjam \
      -p 8009:8009 -p 8109:8109 \
      --env BACKJAM_DATABASE_CONNECTION=mysql
      --env BACKJAM_DATABASE_USER=backjam_user \
      --env BACKJAM_DATABASE_PASSWORD=backjam \
      --env BACKJAM_DATABASE_NAME=backjam_db \
      --network backjam-network \
      --volume /path/to/backjam-data-persistence:/server/data \
      --volume /path/to/backjam-logs-persistence:/server/logs \
      backjam/backjam:latest      
      
## Configuration

### Environment variables

When you start the Backjam image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

- For docker-compose add the variable name and value under the application section in the `docker-compose.yml` file present in this repository:

```
backjam:
  ...
  environment:
    - BACKJAM_DATABASE_PASSWORD=my_password
  ...
```

- For manual execution add a `--env` option with each variable and value:

```
$ docker run -d --name backjam -p 8009:8009 -p 8109:8109 \
 --env BACKJAM_DATABASE_PASSWORD=my_password \
 --network backjam-network \
 --volume /path/to/backjam-data-persistence:/server/data \
 --volume /path/to/backjam-logs-persistence:/server/logs \
 backjam/backjam:latest
```

Available variables:

#### User and Site configuration

- `BACKJAM_ADMIN_USERNAME`: Backjam initial administrator username. Default: **admin**
- `BACKJAM_ADMIN_PASSWORD`: Backjam initial administrator password. Default: **backjam123**
- `BACKJAM_ADMIN_EMAIL`: Backjam initial administrator email. Default: **user@example.com**
- `BACKJAM_WEB_HOST`: Backjam hostname/address for admin UI page. Default: **http://locahost:8109**.
- `BACKJAM_API_HOST`: Backjam hostname/address for API calls. This must be exposed and reachable by the UI page. Default: **http://locahost:8009**.
- `BACKJAM_JWT_ACCESS_TOKEN_SECRET_KEY`: A secret value that will be used to generate JWT access tokens. Default: *empty*
- `BACKJAM_JWT_REFRESH_TOKEN_SECRET_KEY`: A secret value that will be used to generate JWT refresh tokens. Default: *empty*
    
#### Database connection configuration

- `BACKJAM_DATABASE_CONNECTION`: Database to be used. Can be `sqlite` or `mysql`. If `mysql` is provided, the following variables are read. If none is provided, defaults to using an in-memory database.
- `BACKJAM_DATABASE_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `BACKJAM_DATABASE_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `BACKJAM_DATABASE_NAME`: Database name that Backjam will use to connect with the database. Default: **backjam_db**
- `BACKJAM_DATABASE_USER`: Database user that Backjam will use to connect with the database. Default: **backjam_user**
- `BACKJAM_DATABASE_PASSWORD`: Database password that Backjam will use to connect with the database. No defaults.

## Examples

### Connect Backjam container to an existing database

The Backjam container supports connecting the Backjam application to an external database. This would be an example of using an external database for Backjam.

Modify the `docker-compose.yml` file present in this repository:

```
 wordpress:
   ...
   environment:
-      - BACKJAM_DATABASE_HOST=mariadb
+      - BACKJAM_DATABASE_HOST=mariadb_host
     - BACKJAM_DATABASE_PORT_NUMBER=3306
     - BACKJAM_DATABASE_NAME=backjsm_db
     - BACKJAM_DATABASE_USER=backjam_user
+      - BACKJAM_DATABASE_PASSWORD=backjam_password
   ...
```

For manual execution:

```
  $ docker run -d --name backjam \
    -p 8009:8009 -p 8109:8109 \
    --network backjam-network \
    --env BACKJAM_DATABASE_HOST=mariadb_host \
    --env BACKJAM_DATABASE_PORT_NUMBER=3306 \
    --env BACKJAM_DATABASE_NAME=backjam_db \
    --env BACKJAM_DATABASE_USER=backjam_user \
    --env BACKJAM_DATABASE_PASSWORD=backjam_password \
    --volume backjam_data:/server/data \
    --volume backjam_logs:/server/logs \
    backjam/backjam:latest
```


## Logging

The Backjam Docker image sends the container logs to stdout and to `/server/logs`. To view the logs:

    $ docker logs backjam

Or using Docker Compose:

    $ docker-compose logs backjam


## Maintenance

### Backing up your container

To backup your data, configuration and logs, follow these simple steps:

#### Step 1: Stop the currently running container

    $ docker stop backjam

Or using Docker Compose:

    $ docker-compose stop backjam

#### Step 2: Run the backup command

We need to mount two volumes in a container we will use to create the backup: a directory on your host to store the backup in, and the volumes from the container we just stopped so we can access the data.

    $ docker run --rm -v /path/to/backjam-backups:/backups --volumes-from backjam busybox \
      cp -a /server/data /backups/latest

### Restoring a backup

Restoring a backup is as simple as mounting the backup as volumes in the containers.

For the MariaDB database container:

```
 $ docker run -d --name mariadb \
   ...
-  --volume /path/to/mariadb-persistence:/bitnami/mariadb \
+  --volume /path/to/mariadb-backups/latest:/bitnami/mariadb \
   bitnami/mariadb:latest
```

For the Backjam container:

```
 $ docker run -d --name backjam \
   ...
-  --volume /path/to/backjam-persistence:/server/data \
+  --volume /path/to/backjam-backups/latest:/server/data \
   backjam/backjam:latest
   
```

## Upgrade this image

Backjam provides up-to-date versions, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Backjam container. For the MariaDB upgrade see: https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image


#### Step 1: Get the updated image

    $ docker pull backjam/backjam:latest

#### Step 2: Stop the running container

Stop the currently running container using the following command:

    $ docker-compose stop backjam

#### Step 3: Take a snapshot of the application state

Follow the steps in [Backing up your container](#backing-up-your-container) to take a snapshot of the current application state.

### Step 4: Remove the currently running container

Remove the currently running container by executing the following command:

    docker-compose rm -v backjam

#### Step 5: Run the new image

Update the image tag in `docker-compose.yml` and re-create your container with the new image:

    $ docker-compose up -d

## Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/get-backjam/Backjam/issues). For us to provide better support, be sure to include the following information in your issue:

    Host OS and version
    Docker version (docker version)
    Output of docker info
    Version of this container
    The command you used to run the container, and any relevant output you saw (masking any sensitive information)

## License

Copyright (c) 2021 Backjam

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
