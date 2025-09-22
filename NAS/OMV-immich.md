
Source of example 
https://github.com/immich-app/immich/discussions/13124#discussioncomment-11031865

Compose file for OMV to use different user.
```yaml

name: immich

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    user: "${IMMICH_USER}:${IMMICH_USER}"
    # Add group owner-ship of the external folders.
    # group_add:
    #   - $USER1_GID
    volumes:
        # The folder used by IMMICH to save uploaded image.
      - ${POOL1_PICTURES}/immich/uploads:/usr/src/app/upload
      - ${POOL1_PICTURES}/immich/ext:/assets/User1
      - /etc/localtime:/etc/localtime:ro
    env_file:
      # This is to pair with the immich compose name set in OMV
      - immich.env
    ports:
      - 2283:2283
    depends_on:
      - redis
      - database
    restart: unless-stopped
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    user: "${IMMICH_USER}:${IMMICH_USER}"
    # group_add:
    #   - $USER1_GID

    volumes:
      - ${IMMICH_APPDATA}/cache:/cache
      - ${IMMICH_APPDATA}/.config:/.config
    env_file:
      - immich.env
    restart: unless-stopped
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/redis:6.2-alpine@sha256:2ba50e1ac3a0ea17b736ce9db2b0a9f6f8b85d4c27d5f5accc6a416d8f42c6d5
    user: "${IMMICH_USER}:${IMMICH_USER}"
    # group_add:
    #   - $USER1_GID
    volumes:
      - ${IMMICH_APPDATA}/data:/data
    healthcheck:
      test: redis-cli ping || exit 1
    restart: unless-stopped

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:90724186f0a3517cf6914295b5ab410db9ce23190a2d9d0b9dd6463e3fa298f0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    volumes:
      - ${IMMICH_APPDATA}/postgres:/var/lib/postgresql/data
    restart: unless-stopped
```

env file (set in OMV gui)
```yaml


# Base folder for all configs
IMMICH_APPDATA=${APPDATA_PATH}/immich


# The Immich version to use. You can pin this to a specific version like "v1.71.0"
IMMICH_VERSION=release

# Connection secret for postgres. You should change it to a random password
# Please use only the characters `A-Za-z0-9`, without special characters or spaces
DB_PASSWORD=postgres

# The values below this line do not need to be changed
###################################################################################
DB_USERNAME=postgres
DB_DATABASE_NAME=immich

```