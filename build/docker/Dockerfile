# --- Stage 1: PostgreSQL 18 Clients extrahieren (musl-kompatibel) ---
FROM postgres:18.0-alpine AS pg18


# --- Stage 2: Builder ---
FROM python:3.11-alpine AS builder

LABEL MAINTAINER="la-cc@github"
LABEL org.opencontainers.image.source="https://github.com/la-cc/k8s-pg-to-s3-cronjob"
LABEL EMAIL="artem@lajko.dev"

RUN apk add --no-cache \
    openssl \
    zip \
    git \
    p7zip \
    curl \
    groff \
    less \
 && apk add --no-cache --virtual .build-deps \
    gcc \
    musl-dev \
    libffi-dev \
    openssl-dev \
    python3-dev \
    cargo \
 && pip install --no-cache-dir --upgrade pip \
 && pip install --no-cache-dir awscli awscli-plugin-endpoint \
 && apk del .build-deps

# Get PostgreSQL 18 client binaries and libraries from pg18
COPY --from=pg18 /usr/local/bin/psql /usr/local/bin/psql
COPY --from=pg18 /usr/local/bin/pg_dump /usr/local/bin/pg_dump
COPY --from=pg18 /usr/local/bin/pg_dumpall /usr/local/bin/pg_dumpall
COPY --from=pg18 /usr/local/lib/ /usr/local/lib/


# --- Stage 3: Runtime ---
FROM python:3.11-alpine

LABEL MAINTAINER="la-cc@github"
LABEL org.opencontainers.image.source="https://github.com/la-cc/k8s-pg-to-s3-cronjob"
LABEL EMAIL="artem@lajko.dev"

# Set environment variables
## AWS S3
ENV AWS_S3_ENDPOINT_URL=""
ENV AWS_S3_ACCESS_KEY_ID=""
ENV AWS_S3_SECRET_ACCESS_KEY=""
ENV AWS_S3_BUCKET=""

## Postgres
ENV DATABASE_PASSWORD=""
ENV DATABASE_USER=""
ENV DATABASE_HOSTNAME=""
ENV DATABASE_NAMES=""
ENV DATABASE_PORT=""

## Webhook
ENV WEBHOOK_ENDPOINT=""
ENV ENABLE_WEBHOOK_ENDPOINT=true

## Encryption key
ENV ENCRYPTION_KEY=""


# Install runtime dependencies
RUN apk add --no-cache \
    openssl \
    curl \
    zip \
    libstdc++ \
    libgcc \
    lz4-libs \
    openldap \
    libedit \
    zstd-libs

# PostgreSQL 18 Clients take over
COPY --from=pg18 /usr/local/bin/pg_dump /usr/local/bin/pg_dump
COPY --from=pg18 /usr/local/bin/psql /usr/local/bin/psql
COPY --from=pg18 /usr/local/bin/pg_dumpall /usr/local/bin/pg_dumpall
COPY --from=pg18 /usr/local/lib/ /usr/local/lib/

# Take over AWS CLI and Python libs
COPY --from=builder /usr/local/bin/aws /usr/local/bin/aws
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/bin/openssl /usr/bin/openssl
COPY --from=builder /usr/bin/curl /usr/bin/curl
COPY --from=builder /usr/bin/zip /usr/bin/zip
COPY --from=builder /usr/bin/7z /usr/bin/7z


# Create a non-root user 'backupuser'
RUN addgroup -S backupuser && adduser -S backupuser -G backupuser

# Set work directory and switch to non-root user
WORKDIR /home/backupuser
COPY /build/docker/scripts/entrypoint.sh .

USER backupuser

# Adjust the ENTRYPOINT as needed, for now, it's set to keep the container running
ENTRYPOINT [ "./entrypoint.sh", "-f"]

# ENTRYPOINT [ "./entrypoint.sh", "-f" ]
# ENTRYPOINT ["tail", "-f", "/dev/null"]
