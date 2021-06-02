# Promscale
Promscale is an open source long-term store for Prometheus data designed for
analytics. It is built on top of TimescaleDB, and is a horizontally scalable and
operationally mature platform for Prometheus data that uses PromQL and SQL to
allow you to ask any question, create any dashboard, and achieve greater
visibility into your systems.

For more information about Promscale, see our [blog post][promscale-blog], or
check out the [demo][promscale-demo].

## Install Promscale
There are several different methods for installing Promscale. This section
describes installing from a pre-built Docker image. For alternative installation
methods, see the [Promscale GitHub repository][promscale-github].

If you have a Kubernetes cluster with Helm installed, you can use
the observability suite for Kubernetes (tobs) to install a full metric
collection and visualization solution including Prometheus, Grafana, Promscale,
and a preview version of PromLens. To learn how to do this, watch
our [demo video][tobs demo video].

If you want to migrate data from Prometheus into Promscale, you can use
[Prom-migrator][prom-migrator-blog], an open-source, universal Prometheus data
migration tool that can move data from one remote-storage system to another.

### Procedure: Installing Promscale from a Docker image
1.  Download the Docker image for the Promscale Connector
from [Docker Hub][promscale-docker-hub]. For more information about this image,
see the [Promscale Releases on GitHub][promscale-releases-github].
1.  Create a network specific to Promscale-TimescaleDB:
    ```bash
    docker network create --driver bridge promscale-timescaledb
    ```
1.  Install and run TimescaleDB with the Promscale extension:
    ```bash
    docker run --name timescaledb -e \
    POSTGRES_PASSWORD=<password> -d -p 5432:5432 \
    --network promscale-timescaledb \
    timescaledev/promscale-extension:latest-pg12 \
    postgres -csynchronous_commit=off
    ```
1.  Run Promscale:
    ```bash
    docker run --name promscale -d -p 9201:9201 \
    --network promscale-timescaledb timescale/promscale:<version-tag> \
    -db-password=<password> -db-port=5432 -db-name=postgres \
    -db-host=timescaledb -db-ssl-mode=allow
    ```
    In this example, we use `db-ssl-mode=allow`, which is suitable for testing
    purposes. For production environments, use `db-ssl-mode=require` instead.


## Docker maintenance tasks
Docker installations need to run a maintenance procedure regularly. This
procedure performs necessary maintenance tasks like enforcing data retention
policies. We recommend that you set up a cron job to run a job like this every
thirty minutes:

```bash
docker exec \
  --user postgres \
  timescaledb \
    psql \
      -c "CALL execute_maintenance();"
```

## Configure Prometheus for Promscale
You need to tell Prometheus to use the remote storage connector. By setting
Prometheus to `read_recent` it means that Prometheus queries data from Promscale
for all PromQL queries. You can do that by opening the `prometheus.yml`
configuration file, and adding these lines:
```yaml
remote_write:
  - url: "http://<connector-address>:9201/write"
remote_read:
  - url: "http://<connector-address>:9201/read"
    read_recent: true
```

For more information about configuring Prometheus for Promscale, see the [Promscale documentation][promscale-config-github].

## Configure the Promscale Connector
You can configure the Promscale Connector using flags at the command prompt, environment variables, or a YAML configuration file. When processing commands, precedence is granted in this order:
1. CLI flag
1. Environment variable
1. Configuration file value
1. Default value

All environment variables are prefixed with `PROMSCALE`.

For more information about configuring Promscale, see the [Promscale CLI documentation][promscale-cli-github], or use the `promscale -h` command.


[promscale-blog]: https://blog.timescale.com/blog/promscale-analytical-platform-long-term-store-for-prometheus-combined-sql-promql-postgresql/
[promscale-demo]: https://youtu.be/FWZju1De5lc
[tobs demo video]: https://youtu.be/MSvBsXOI1ks
[prom-migrator-blog]: https://blog.timescale.com/blog/introducing-prom-migrator-a-universal-open-source-prometheus-data-migration-tool/
[promscale-github]: https://github.com/timescale/promscale
[promscale-docker-hub]: https://hub.docker.com/r/timescale/promscale/
[promscale-releases-github]: https://github.com/timescale/promscale/releases
[promscale-config-github]: https://github.com/timescale/promscale/blob/master/docs/configuring_prometheus.md
[promscale-cli-github]: https://github.com/timescale/promscale/blob/master/docs/cli.md