# CronJob: PostgreSQL Backup to S3-Compatible Object Storage with Encryption and Webhook Notification

This README explains how to use the provided Dockerfile and shell script to back up databases to any S3-compatible object storage, with encryption and webhook notifications for each successful upload.

## Preparation Steps

WIP!!!
DIAGRAM TBD!!!

1. Create a user in the S3-compatible object storage service.
2. Configure the AWS CLI with the user's access key and secret key.
3. Create an S3 bucket with object lock enabled.
4. Configure object lock for the bucket.
5. Verify the object lock configuration.
6. View the object lock retention configuration.
7. Set up a webhook endpoint for notifications.
8. Create PostgreSQL databases that should be backup up.

## Overview

The Dockerfile builds an image incorporating the necessary tools for database backup, file encryption, and performing webhook notifications. The accompanying shell script executes the following operations:

1. Generates a dump of specified PostgreSQL databases.
2. Encrypts and compresses each dump file using a predefined encryption key.
3. Uploads the encrypted files to an S3-compatible object storage service.
4. Notifies a configured webhook endpoint about each successful upload.

## Requirements

- Docker
- Accessible PostgreSQL databases that should be backed up
- A server or Kubernetes cluster for deploying the CronJob (optional)
- A webhook endpoint for upload notifications

## Configuration

Sensitive details such as database credentials, object storage access keys, and encryption keys should be securely managed. When deployed within a Kubernetes environment, it's advisable to utilize secrets for injecting these values at runtime.

### Environment Variables

- `AWS_S3_ENDPOINT_URL`: Endpoint URL for the S3-compatible object storage.
- `AWS_S3_ACCESS_KEY_ID`: Access key ID with permissions to the object storage bucket.
- `AWS_S3_SECRET_ACCESS_KEY`: Secret access key for the object storage.
- `AWS_S3_BUCKET`: Name of the object storage bucket for storing backups.
- `DATABASE_PASSWORD`: Password for the PostgreSQL database user.
- `DATABASE_USER`: Username for accessing the PostgreSQL database.
- `DATABASE_HOSTNAME`: Hostname of the PostgreSQL database server.
- `DATABASE_NAMES`: Comma-separated list of databases to back up.
- `DATABASE_PORT`: Port for the PostgreSQL database server.
- `WEBHOOK_ENDPOINT`: URL of the webhook to notify upon successful backup upload.
- `ENCRYPTION_KEY`: Key for encrypting backup files.

## Deployment Instructions

1. **Build the Docker Image:**

```sh
docker \
    build . \
    -f build/docker/Dockerfile \
    -t ghcr.io/la-cc/k8s-pg-s3-cronjob:0.0.0
```

2. **Run the Container Locally (For Testing):**

```sh
docker run --env-file .env ghcr.io/la-cc/k8s-pg-s3-cronjob:0.0.0
```

`.env` is a file that contains all necessary environment variables.

3. **Kubernetes Deployment:**

WIP!!!

For a Kubernetes deployment, create a `CronJob` resource, along with `Secret` and `ConfigMap` resources, to manage the environment variables and backup schedule.

### Kubernetes Resources

- **Secret**: Stores sensitive information like object storage credentials and database password.
- **ConfigMap**: Manages non-sensitive configuration like database names, ports, and object storage endpoint.
- **CronJob**: Automates the backup process based on a defined schedule.

Consult the Kubernetes documentation to create these resources according to the specified environment variables.

## Security

The Dockerfile configures a non-root user (`backupuser`) to enhance container security. Ensure the Kubernetes `CronJob` specification includes an appropriate `securityContext` to respect this setup.

## Conclusion

This configuration offers a secure, automated way to back up PostgreSQL databases to any S3-compatible object storage service, with notifications upon successful uploads. Customize the setup as needed to align with your specific environment and security protocols.
