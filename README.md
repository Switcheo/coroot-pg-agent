# SET UP 

## Navigate to `postgresql.conf`

- Uncomment these lines
- Enable tracking
- add temp directory and shared preload libraries

```bash
[STATISTICS]
# - Query and Index Statistics Collector -

track_activities = on
track_activity_query_size = 1024        # (change requires restart)
track_counts = on
track_io_timing = on
track_wal_io_timing = on
track_functions = all                   # none, pl, all
stats_temp_directory = 'pg_stat_tmp'

[CLIENT CONNECTION DEFAULTS]

# - Shared Library Preloading -
shared_preload_libraries = 'pg_stat_statements' # (change requires restart)
```

## Restart postgresql

```bash
brew services restart postgresql
```

## Create extension

Connect to carbon db

```sql
psql carbon
```

Create extension

```sql
CREATE EXTENSION pg_stat_statements;
```

## Build the docker image
1. Clone the repo
    ```shell
    git clone https://github.com/Switcheo/coroot-pg-agent.git
    ```
2. Build the docker image
    ```shell
    docker build -t <image tag> .
   ```

## Run the program using docker

```bash
docker run --detach --name coroot-pg-agent \
--env DSN="postgres://<user>@host.docker.internal:5432/carbon?connect_timeout=1&statement_timeout=30000&sslmode=disable" \
--env LISTEN="0.0.0.0:<port>" \
--env PG_SCRAPE_INTERVAL=1s \
-p <port>:<port>\
<image tag>
```

### Issues:

- **psql Connection refused**
- **Cannot assign requested address**

### Solution:

[Dockerizing PostgreSQL - psql Connection refused](https://stackoverflow.com/a/58081948)

https://github.com/npgsql/efcore.pg/issues/225

- `host.docker.internal` should be used on windows and mac
- `localhost` should be used on linux

## Add target to prometheus

- In `prometheus.yml` under `scrape_configs:`

    ```bash
    - job_name: "postgres"
    	static_configs:
    		- targets: [ "localhost:<port>" ]
    ```


### Example:
```bash
docker run --detach --name coroot-pg-agent \
--env DSN="postgres://nicholas@host.docker.internal:5432/carbon?connect_timeout=1&statement_timeout=30000&sslmode=disable" \
--env LISTEN="0.0.0.0:3002" \
--env PG_SCRAPE_INTERVAL=1s \
-p 3002:3002 \
ghcr.io/coroot/coroot-pg-agent
```

```bash
- job_name: "postgres"
	static_configs:
		- targets: [ "localhost:3002" ]
```