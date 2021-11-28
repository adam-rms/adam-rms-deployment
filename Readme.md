# AdamRMS Deployment

This docker-compose file is an example of how you could deploy [AdamRMS](https://github.com/bstudios/adam-rms) on a production server

# Server Setup

## Docker

1. Install docker `apt install docker.io docker-compose`
1. Ensure docker boots on startup (`systemctl enable docker`)
1. Clone this repo `git clone https://github.com/bstudios/adam-rms-deployment.git`
1. `cd adam-rms-deployment` to get into the correct folder
1. Download the Cloudflare Origin Certificate - place the `ssl.crt`, `ssl.key` & `origin-pull-ca.pem` files in the root of the directory
1. Create `.env` based on the example file, and fill out the details (do this with `nano .env`)
1. Run `docker-compose up -d` to start the stack!
1. Run `docker stats` to confirm it's running okay

## Container Bash

To enable you to access the AdamRMS container to administer it 

`docker exec -t -i adamrms /bin/bash`

## Updating

Watchtower does the updating for you anytime the docker image for AdamRMS is updated (which triggers a docker build) but if you update this repo you'll need to run `git pull && docker-compose up -d`

## Database

## MySQL Setup

To generate a MySQL dump file run: 

```bash
docker exec -i db /usr/bin/mysqldump -u root --password=ROOTPASSWORD adamrms > backup.sql
```

To transfer a MySQL dump file (named `backup.sql`) run:

```bash
cat backup.sql | docker exec -i db /usr/bin/mysql -u root --password=ROOTPASSWORD adamrms
```

## MySQL Backups

`mysql-backup` runs a backup every hour and dumps it in S3. Ensure the bucket has an appropriate lifecycle rule setup

# File Storage

_This section requires further documentation_

## Backblaze B2

### Bucket CORS 

You need to setup the CORS on the bucket to allow uploads

1. First download and authenticate the AWS Cli (using your Backblaze credentials), then download the cors.json file from this repo and run:
2. `aws s3api put-bucket-cors --bucket=BUCKETNAME --endpoint-url=https://s3.eu-central-003.backblazeb2.com  --cors-configuration=file://backblaze-cors.json`

### Bucket Info

Set the bucket info to:

`{"cache-control":"public, max-age=900, s-maxage=3600, stale-while-revalidate=900, stale-if-error=3600"}`

### Config

Then populate your .env file with config: 

```
bCMS__AWS_S3_BUCKET_REGION=eu-central-003
bCMS__AWS_S3_BUCKET_NAME=bucket-name
bCMS__AWS_SERVER_KEY=00xxxxx00
bCMS__AWS_SERVER_SECRET_KEY=K0000000000000/0000000000000
bCMS__AWS_S3_CDN=https://s3.eu-central-003.backblazeb2.com/bucket-name
bCMS__AWS_S3_BUCKET_ENDPOINT=s3.eu-central-003.backblazeb2.com
```

## AWS S3

To use AWS S3 you need three seperate IAM users.

### PHP Server for Uploads

Set these credentials in:
```
bCMS__AWS_S3_BUCKET_REGION=eu-west-1
bCMS__AWS_S3_BUCKET_NAME=adamrms-userfiles
bCMS__AWS_SERVER_KEY=
bCMS__AWS_SERVER_SECRET_KEY=
bCMS__AWS_S3_CDN=https://cdn.adam-rms.com
bCMS__AWS_S3_BUCKET_ENDPOINT=s3.eu-west-1.amazonaws.com
```

Policy Required

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::BUCKETNAME/*"
        }
    ]
}
```

### Cloudfront keys for downloads

Set these credentials in:
```
bCMS__AWS_ACCOUNT_CLOUDFRONT_ENABLED=TRUE
bCMS__AWS_ACCOUNT_PRIVATE_KEY=
bCMS__AWS_ACCOUNT_PRIVATE_KEY_ID=
bCMS__AWS_S3_CDN_CLOUDFRONT=https://cdn.adam-rms.com
```

Note that you obtain these credentials from the cloudfront dashboard, not IAM.

Also note that when setting up a policy for the Cloudfront distribution, you must enable: 

- Query strings - Include specified query strings `response-content-disposition`

### Database backups 

Set these credentials in:
```
AWS_ENDPOINT_URL=https://s3.eu-west-1.amazonaws.com
DB_DUMP_TARGET=s3://BUCKETNAME
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

The database backup IAM role requires two policies:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1584879011000",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:ListMultipartUploadParts",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:PutObjectTagging",
                "s3:PutObjectVersionAcl",
                "s3:PutObjectVersionTagging"
            ],
            "Resource": [
                "arn:aws:s3:::BUCKETNAME/*"
            ]
        }
    ]
}
```