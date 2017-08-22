# Production Docker deployment for Mattermost

This project enables deployment of a Mattermost server in a multi-node production configuration using Docker.

[![Build Status](https://travis-ci.org/mattermost/mattermost-docker.svg?branch=master)](https://travis-ci.org/mattermost/mattermost-docker)

Notes:
- The default Mattermost edition for this repo has changed from Team Edition to Enterprise Edition. Please see [Choose Edition](#choose-edition-to-install) section.
- To install this Docker project on AWS Elastic Beanstalk please see [AWS Elastic Beanstalk Guide](contrib/aws/README.md).
- To run Mattermost on Kubernetes you can start with the [manifest examples in the kubernetes folder](contrib/kubernetes/README.md)
- To install Mattermost without Docker directly onto a Linux-based operating systems, please see [Admin Guide](https://docs.mattermost.com/guides/administrator.html#installing-mattermost).

## Installation using Docker Compose

The following instructions deploy Mattermost in a production configuration using multi-node Docker Compose set up.

### Requirements

* [docker]
* [docker-compose]

### Choose Edition to Install

If you want to install Enterprise Edition, you can skip this section.

To install the Team Edition, comment out the following line in docker-compose.yaml file:
```
dockerfile: Dockerfile-enterprise
```

### Database container
This repository offer a Docker image for the Mattermost database. It is a customized PostgreSQL image that you should configure with following environment variables :
* `POSTGRES_USER`: database username
* `POSTGRES_PASSWORD`: database password
* `POSTGRES_DB`: database name

#### AWS
If deploying to AWS, you could also set following variables to enable [Wal-E](https://github.com/wal-e/wal-e) backup to S3 :
* `AWS_ACCESS_KEY_ID`: AWS access key
* `AWS_SECRET_ACCESS_KEY`: AWS secret
* `WALE_S3_PREFIX`: AWS s3 bucket name
* `AWS_REGION`: AWS region

All four environment variables are required. It will enable completed WAL segments sent to archive storage (S3). The base backup and clean up can be done through the following command:
```bash
# Base backup
docker exec mattermost-db su - postgres sh -c "/usr/bin/envdir /etc/wal-e.d/env /usr/local/bin/wal-e backup-push /var/lib/postgresql/data"
# Keep the most recent 7 base backups and remove the old ones
docker exec mattermost-db su - postgres sh -c "/usr/bin/envdir /etc/wal-e.d/env /usr/local/bin/wal-e delete --confirm retain 7"
```
Those tasks can be executed through a cron job or systemd timer.

### Application container
Application container run the Mattermost application. You should configure it with following environment variables :
* `MM_USERNAME`: database username
* `MM_PASSWORD`: database password
* `MM_DBNAME`: database name

If your database use some custom host and port, it is also possible to configure them :
* `DB_HOST`: database host address
* `DB_PORT_NUMBER`: database port

If you use a Mattermost configuration file on a different location than the default one (`/mattermost/config/config.json`) :
* `MM_CONFIG`: configuration file location inside the container.

If you choose to use MySQL instead of PostgreSQL, you should set a different datasource :
* `MM_SQLSETTINGS_DATASOURCE` : `"$MM_USERNAME:$MM_PASSWORD@tcp($DB_HOST:$DB_PORT_NUMBER)/$MM_DBNAME?charset=utf8mb4,utf8&readTimeout=30s&writeTimeout=30s"`

### Web server container
This image is optional, you should not use it you have your own reverse-proxy. It is a simple front Web server for the Mattermost app container.
* `MATTERMOST_ENABLE_SSL`: whether to enable SSL
* `PLATFORM_PORT_80_TCP_PORT`: port that Mattermost image is listening on

#### Install with SSL certificate
Put your SSL certificate as `./volumes/web/cert/cert.pem` and the private key that has
no password as `./volumes/web/cert/key-no-password.pem`. If you don't have
them you may generate a self-signed SSL certificate.

## Starting/Stopping

### Start
```
docker-compose start
```

### Stop
```
docker-compose stop
```

### Update

Make sure to backup Mattermost data before proceeding.
```
docker-compose down
git pull
docker-compose build
docker-compose up -d
```

## Removing

### Remove the containers
```
docker-compose stop && docker-compose rm
```

### Remove the data and settings of your Mattermost instance
```
sudo rm -rf volumes
```

## Upgrading to Team Edition 3.0.x from 2.x

You need to migrate your database before upgrading Mattermost to `3.0.x` from
`2.x`. Run these commands in the latest `mattermost-docker` directory.
```
docker-compose rm -f app
docker-compose build app
docker-compose run app -upgrade_db_30
docker-compose up -d
```
See the [offical Upgrade Guide](http://docs.mattermost.com/administration/upgrade.html) for more details.

## Known Issues

* Do not modify the Listen Address in Service Settings.
* Rarely `app` container fails to start because of "connection refused" to
  database. Workaround: Restart the container.

## More information

If you want to know how to use docker-compose, see [the overview
page](https://docs.docker.com/compose).

For the server configurations, see [prod-ubuntu.rst] of Mattermost.

[docker]: http://docs.docker.com/engine/installation/
[docker-compose]: https://docs.docker.com/compose/install/
[prod-ubuntu.rst]: https://docs.mattermost.com/install/install-ubuntu-1404.html
