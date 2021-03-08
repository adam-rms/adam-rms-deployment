# AdamRMS Deployment

This docker-compose file allows the AdamRMS Website to be deployed on production servers.

# Server Setup

## Docker

1. `systemctl enable docker` to ensure docker boots on startup
1. Login to Github docker container registry `docker login ghcr.io` and ensure that it creates a config file for you (located at `/root/.docker/config.json`)
1. Clone the repo `git clone git@github.com:bstudios/adam-rms-deployment.git`
1. `cd website` to get into it
1. Download the Cloudflare Origin Certificate - place the `ssl.crt`, `ssl.key` & `origin-pull-ca.pem` files in the root of the directory
1. Create `.env` based on the example file, and fill out the details (do this with `nano .env`)
1. Run `docker-compose up -d` to run

## Container Bash

`docker exec -t -i adamrms /bin/bash`

## Updating

Watchtower does the updating for you anytime you push a new tag (which triggers a docker build) but if you update this repo you'll need to run `git pull && docker-compose up -d`

## Database

## MySQL Setup

To transfer a MySQL dump file (named `backup.sql`) run:
```
cat backup.sql | docker exec -i db /usr/bin/mysql -u root --password=MYSqlRootPassword nouse
```

## MySQL Backups

`mysql-backup` runs a backup every day at about 2:30am and dumps it in S3. Ensure the bucket has an appropriate lifecycle rule setup