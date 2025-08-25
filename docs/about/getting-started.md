---
title: Getting Started
layout: home
nav_order: 2
---

# Getting started

**Access via**: [http://localhost:36400](http://localhost:36400) (user = admin, password = admin)

To ensure the data is saved across builds, link an empty volume to: `/var/www/config` within the container. This is where the `env` file will be stored, along with the sqlite database and the application log files.

Note about websocket
{: .label .label-purple }

**NOTE**: You will need to set the `REVERB_HOST` variable to the machine IP where **m3u editor** is runing for websockets to work correctly (if not running on `localhost`). 

#### Versions

We inlcude both **linux/amd64** and **linux/arm64** builds in our build process, starting at version `v0.4.5`. We also push regular updates and new beta features to the `experimental` and `dev` branches.

- 💪 Use the latest stable release: `sparkison/m3u-editor:latest`
- #️⃣ Use a specific version: `sparkison/m3u-editor:0.6.4`, `sparkison/m3u-editor:0.6.8`, etc.
- 🔥 Use the dev branch: `sparkison/m3u-editor:dev`
  - stable'ish branch – we try to keep this one clean and fix bugs as quickly as possible
- 🧪 Use the experimental branch: `sparkison/m3u-editor:experimental`
  - new and fun features that can break things; features will be announced on our Discord under the **#releases** channel

#### Table of contents

- [Docker compose (simplest)](#-docker-compose)
- [Docker with internal Postgres (recommended)](#-docker-compose-with--sql-postgresql) <sup>(v0.6.0+)</sup>
- [Docker with your Postgres (advanced)](#-if-youd-like-to-use-your-own-postgresql-instance) <sup>(v0.6.0+)</sup>
- [Notes](#-notes)
---

## 🐳 Docker compose

Use the following compose example to get up and running.

```yaml
services:
  m3u-editor:
    image: sparkison/m3u-editor:latest
    container_name: m3u-editor
    environment:
      - TZ=Etc/UTC
      - APP_URL=http://localhost # or http://192.168.0.123 or https://your-custom-tld.com
      # This is used for websockets and in-app notifications
      # Set to your machine/container IP where m3u editor will be accessed, if not localhost
      - REVERB_HOST=localhost # or 192.168.0.123 or your-custom-tld.com
      - REVERB_SCHEME=http # or https if using custom TLD with https
    volumes:
      # This will allow you to reuse the data across container recreates
      # Format is: <host_directory_path>:<container_path>
      # More information: https://docs.docker.com/reference/compose-file/volumes/
      - ./data:/var/www/config
      # Optionally, for optimal performance using HLS proxy, link a Ram Disk to the HLS file location used for transoding
      # - /mnt/RamDisk:/var/www/html/storage/app/hls
    restart: unless-stopped
    ports:
      - 36400:36400 # app
      - 36800:36800 # websockets/broadcasting
networks: {}
```

Or via Docker CLI:

```bash
 docker run --name m3u-editor -e TZ=Etc/UTC -e APP_URL=http://localhost -e REVERB_HOST=localhost -e REVERB_SCHEME=http -v ./data:/var/www/config --restart unless-stopped -p 36400:36400 -p 36800:36800 sparkison/m3u-editor:latest 
```

---

## 🐳 Docker compose with 🐘 SQL (PostgreSQL)
{: .d-inline-block }

New (v0.6.0)
{: .d-inline-block .v-align-text-bottom .label .label-purple }

By default **m3u editor** uses **SQLite** as the database driver. 

If you'd like something more resilient, you can switch to the **pgsql** driver instead and utilize the internal **PostgreSQL** instance for your database storage. **m3u editor** supports this out of the box!

Update your `docker-compose.yaml` file like this to use **SQL/PostgreSQL**:

```yaml
services:
  m3u-editor:
    image: sparkison/m3u-editor:latest
    container_name: m3u-editor
    environment:
      - TZ=Etc/UTC
      - APP_URL=http://localhost # or http://192.168.0.123 or https://your-custom-tld.com
      # This is used for websockets and in-app notifications
      # Set to your machine/container IP where m3u editor will be accessed, if not localhost
      - REVERB_HOST=localhost # or 192.168.0.123 or your-custom-tld.com
      - REVERB_SCHEME=http # or https if using custom TLD with https
      - ENABLE_POSTGRES=true               # <----- start here
      - PG_DATABASE=${PG_DATABASE:-m3ue}   # <----- DB name
      - PG_USER=${PG_USER:-m3ue}           # <----- DB user
      - PG_PASSWORD=${PG_PASSWORD:-secret} # <----- DB password
      - PG_PORT=${PG_PORT:-5432}           # <----- DB port (optional, defaults to 5432)
      - DB_CONNECTION=pgsql                # <----- set to `pgsql` (default is `sqlite`)
      - DB_HOST=localhost                  # <----- make sure set to `localhost`
      - DB_PORT=${PG_PORT:-5432}           # <----- using default Postgres port
      - DB_DATABASE=${PG_DATABASE:-m3ue}   # <----- should match `PG_DATABASE`
      - DB_USERNAME=${PG_USER:-m3ue}       # <----- should match `PG_USER`
      - DB_PASSWORD=${PG_PASSWORD:-secret} # <----- should match `PG_PASSWORD`
    volumes:
      # This will allow you to reuse the data across container recreates
      # Format is: <host_directory_path>:<container_path>
      # More information: https://docs.docker.com/reference/compose-file/volumes/
      - ./data:/var/www/config
      # Optionally, for optimal performance using HLS proxy, link a Ram Disk to the HLS file location used for transoding
      # - /mnt/RamDisk:/var/www/html/storage/app/hls
      - pgdata:/var/lib/postgresql/data # <----- link volume `pgsql` data to retain data
    restart: unless-stopped
    ports:
      - 36400:36400 # app
      - 36800:36800 # websockets/broadcasting
      # - 5432:5432 # <----- (optionally) expose the pgqsl instance
networks: {}
volumes:
  pgdata:           # <----- created named volume for Postgres store
```

then make sure to add `.env` variables in the root of project (where your `docker-compose.yaml` lives) if you want to override the default values, e.g.:

```
PG_DATABASE=database_name
PG_USER=database_user
PG_PASSWORD=a_password
PG_PORT=65432
```

---

## 🔧 If you'd like to use your own PostgreSQL instance
{: .d-inline-block }

New (v0.6.0)
{: .d-inline-block .v-align-text-bottom .label .label-purple }

Just update the `DB_` variables and exclude the `PG_` variables.

```yaml
services:
  m3u-editor:
    image: sparkison/m3u-editor:latest
    container_name: m3u-editor
    environment:
      - TZ=Etc/UTC
      - APP_URL=http://localhost # or http://192.168.0.123 or https://your-custom-tld.com
      # This is used for websockets and in-app notifications
      # Set to your machine/container IP where m3u editor will be accessed, if not localhost
      - REVERB_HOST=localhost # or 192.168.0.123 or your-custom-tld.com
      - REVERB_SCHEME=http # or https if using custom TLD with https
      # - ENABLE_POSTGRES=false     # <----- disable, or exclude variable, either works
      - DB_CONNECTION=pgsql         # <----- set to `pgsql` (default is `sqlite`)
      - DB_HOST=hostname            # <----- your Postgres instance hostname (localhost, 192.168.0.456, etc.)
      - DB_PORT=5432                # <----- your Postgres instance port
      - DB_DATABASE=database        # <----- your Postgres database name
      - DB_USERNAME=user            # <----- user for your Postgres database
      - DB_PASSWORD=password        # <----- password for your Postgres database
    volumes:
      # This will allow you to reuse the data across container recreates
      # Format is: <host_directory_path>:<container_path>
      # More information: https://docs.docker.com/reference/compose-file/volumes/
      - ./data:/var/www/config
      # Optionally, for optimal performance using HLS proxy, link a Ram Disk to the HLS file location used for transoding
      # - /mnt/RamDisk:/var/www/html/storage/app/hls
    restart: unless-stopped
    ports:
      - 36400:36400 # app
      - 36800:36800 # websockets/broadcasting
networks: {}
```

---

## 📕 Notes

### 🩺 Health check options

**m3u editor** has a built-in health check you can use when needed.

as an example, you can add this to your **m3u-editor** docker compose to utilize it:
```yaml
services:
  m3u-editor:
      healthcheck:
        test: ["CMD", "curl", "-f", "http://localhost:36400/up"] # Make sure to update the port if you've changed it, url can remain localhost as it's an internally run command
        interval: 10s
        timeout: 5s
        retries: 5
        start_period: 20s
```

A more complete example would look something like this:

```yaml
services:
  m3u-editor:
    image: sparkison/m3u-editor:latest
    container_name: m3u-editor
    environment:
      - TZ=Etc/UTC
      - APP_URL=http://localhost
      - REVERB_HOST=localhost
      - REVERB_SCHEME=http
      - DB_CONNECTION=pgsql
      - DB_HOST=hostname
      - DB_PORT=5432
      - DB_DATABASE=database
      - DB_USERNAME=user
      - DB_PASSWORD=password
    volumes:
      - ./data:/var/www/config
      # - /mnt/RamDisk:/var/www/html/storage/app/hls
    restart: unless-stopped
    ports:
      - 36400:36400 # app (Nginx/Laravel)
      - 36800:36800 # websockets/broadcasting
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:36400/up"] # Make sure to update the port if you've changed it, url can remain localhost as it's an internally run command
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    depends_on:
      m3u-editor:
        condition: service_healthy
    ports:
      - 8096:8096
    volumes:
      - ./jellyfin/config:/config
      - ./jellyfin/cache:/cache
      - ./jellyfin/media:/media
    restart: unless-stopped

networks: {}
```

### Proxy storage behavior

- **TS proxy streaming** uses **Redis (in-memory only)** to buffer transport stream data. This means no disk writes are involved for TS playback, keeping things fast and ephemeral.  
- **HLS proxy streaming** writes `.ts` segment files to **disk** (default path: `/var/www/html/storage/app/hls` inside the container). This ensures segments are available for clients during the HLS playlist cycle.

---

### ⚡️ Performance tip ⚡️ Use a RamDisk for HLS

By default, HLS segment files are written to your container’s local disk. While this works fine, it can:  
- Increase disk wear (lots of small writes).  
- Cause slower performance on slower storage (e.g., HDD, network drives).  

For optimal performance, you can mount a **RamDisk** to the HLS storage path. This keeps all HLS segments in memory while still making them available to the application.

Example for Linux
{: .label .label-purple }

1. Create a RamDisk mount point on your host (adjust **size=512M** as needed for your stream workload):
```bash
   sudo mkdir -p /mnt/RamDisk
   sudo mount -t tmpfs -o size=512M tmpfs /mnt/RamDisk
```
2. Update your docker-compose.yaml:
```yaml
volumes:
  - /mnt/RamDisk:/var/www/html/storage/app/hls
```
3. Restart your container

If using WSL2 or Docker Desktop
{: .label .label-purple }

You can emulate a RamDisk with a tool like ImDisk or mount to /tmp inside WSL. Point the Docker volume to that path.

Optional setup
{: .label .label-orange }

- Using a RamDisk is optional. If you don’t configure one, HLS will still work (segments just go to disk).
- For smaller deployments or casual use, the default disk-based setup is perfectly fine.
- For heavy HLS workloads or long-running live streams, using a RamDisk is strongly recommended.