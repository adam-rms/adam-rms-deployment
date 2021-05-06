# AdamRMS Deployment

This docker-compose file is an example of how you could deploy [AdamRMS](https://github.com/bstudios/adam-rms) on a production server

# Server Setup

## Docker

1. Ensure docker boots on startup (`systemctl enable docker`)
1. Clone this repo `git clone git@github.com:bstudios/adam-rms-deployment.git`
1. `cd website` to get into it
1. Download the Cloudflare Origin Certificate - place the `ssl.crt`, `ssl.key` & `origin-pull-ca.pem` files in the root of the directory
1. Create `.env` based on the example file, and fill out the details (do this with `nano .env`)
1. Run `docker-compose up -d` to start the stack!

## Container Bash

To enable you to access the AdamRMS container to administer it 

`docker exec -t -i adamrms /bin/bash`

## Updating

Watchtower does the updating for you anytime the docker image for AdamRMS is updated (which triggers a docker build) but if you update this repo you'll need to run `git pull && docker-compose up -d`

## Database

## MySQL Setup

To transfer a MySQL dump file (named `backup.sql`) run:
```
cat backup.sql | docker exec -i db /usr/bin/mysql -u root --password=MYSqlRootPassword nouse
```

## MySQL Backups

`mysql-backup` runs a backup every day at about 2:30am and dumps it in S3. Ensure the bucket has an appropriate lifecycle rule setup


# Backblaze

You need to setup the CORS on the bucket to allow uploads

First download and authenticate the AWS Cli (using your Backblaze credentials), then download the cors.json file from this repo and run:

`aws s3api put-bucket-cors --bucket=BUCKETNAME --endpoint-url=https://s3.eu-central-003.backblazeb2.com  --cors-configuration=file://backblaze-cors.json`
