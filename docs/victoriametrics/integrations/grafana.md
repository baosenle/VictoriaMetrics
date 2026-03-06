---
title: Grafana
weight: 1
menu:
  docs:
    parent: "integrations-vm"
    identifier: "integrations-grafana-vm"
    weight: 1
---

VictoriaMetrics integrates with Grafana using either [Prometheus datasource](https://grafana.com/docs/grafana/latest/datasources/prometheus/)
or [VictoriaMetrics datasource](https://grafana.com/grafana/plugins/victoriametrics-metrics-datasource/) plugins.

Resources:
* [VictoriaMetrics Grafana demo playground](https://play-grafana.victoriametrics.com)
* [VictoriaMetrics and Grafana in docker-compose environment](https://github.com/VictoriaMetrics/VictoriaMetrics/tree/master/deployment/docker#docker-compose-environment-for-victoriametrics)

## VictoriaMetrics datasource

Create [VictoriaMetrics datasource](https://grafana.com/grafana/plugins/victoriametrics-metrics-datasource/)
in Grafana with the following URL for single-server:
```

http://<victoriametrics-addr>:8428
```
_Replace `<victoriametrics-addr>` with the VictoriaMetrics hostname or IP address._

For the cluster version, use `vmselect` address:
```
http://<vmselect-addr>:8481/select/<tenant>/prometheus
```
_Replace `<vmselect-addr>` with the hostname or IP address of vmselect service._ 

If you have more than 1 vmselect, configure [load-balancing](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/#cluster-setup).
Replace `<tenant>` based on your [multitenancy settings](https://docs.victoriametrics.com/victoriametrics/cluster-victoriametrics/#multitenancy).

Once connected, you can start building graphs and dashboards using [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/)
or [MetricsQL](https://docs.victoriametrics.com/metricsql/).

VictoriaMetrics datasource is publicly available on [GitHub](https://github.com/VictoriaMetrics/victoriametrics-datasource).
See more in [plugin docs](https://docs.victoriametrics.com/victoriametrics/victoriametrics-datasource/).

_Creating a datasource may require [specific permissions](https://grafana.com/docs/grafana/latest/administration/data-source-management/).
If you don't see an option to create a data source - try contacting system administrator._


## Prometheus datasource

Create [Prometheus datasource](https://grafana.com/docs/grafana/latest/datasources/prometheus/configure-prometheus-data-source/)
in Grafana. Follow the same connection instructions as for [VictoriaMetrics datasource](#VictoriaMetrics-datasource).

In the "Type and version" section set the type to "Prometheus" and the version to at least "2.24.x".
This allows Grafana to use a more efficient API to get label values:

![Datasource](grafana-datasource-prometheus.webp)

Once connected, you can build graphs and dashboards using [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/).

_Creating a datasource may require [specific permissions](https://grafana.com/docs/grafana/latest/administration/data-source-management/).
If you don't see an option to create a data source - try contacting system administrator._

## AI integration via mcp-grafana and Claude Code

[mcp-grafana](https://github.com/grafana/mcp-grafana) is a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server for Grafana.
It allows AI assistants such as [Claude Code](https://code.claude.com/) to programmatically search dashboards,
query datasources (including VictoriaMetrics), and manage Grafana resources through natural language.

### Prerequisites

* Grafana v9 or newer with API access enabled.
* A [Grafana Service Account Token](https://grafana.com/docs/grafana/latest/administration/service-accounts/) with at least `Viewer` permissions.
* VictoriaMetrics configured as a Grafana datasource (see sections above).
* `mcp-grafana` binary or Docker image.

### Getting mcp-grafana

Download the latest binary from the [mcp-grafana releases page](https://github.com/grafana/mcp-grafana/releases),
or pull the Docker image:

```sh
docker pull ghcr.io/grafana/mcp-grafana:latest
```

### Option 1: Claude Desktop configuration

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or
`%APPDATA%\Claude\claude_desktop_config.json` (Windows) and add the `grafana` server block:

```json
{
  "mcpServers": {
    "grafana": {
      "command": "mcp-grafana",
      "args": [],
      "env": {
        "GRAFANA_URL": "http://localhost:3000",
        "GRAFANA_SERVICE_ACCOUNT_TOKEN": "<your-service-account-token>"
      }
    }
  }
}
```

Replace `GRAFANA_URL` with your Grafana instance address and `GRAFANA_SERVICE_ACCOUNT_TOKEN` with the token you created.
If the binary is not in your `PATH`, provide its full path as the `command` value.
Restart Claude Desktop after saving the file.

### Option 2: Claude Code CLI configuration

Register the MCP server directly from the terminal:

```sh
claude mcp add grafana \
  -e GRAFANA_URL=http://localhost:3000 \
  -e GRAFANA_SERVICE_ACCOUNT_TOKEN=<your-service-account-token> \
  -- mcp-grafana
```

Verify the server is registered:

```sh
claude mcp list
```

### Option 3: Docker-based setup

Run `mcp-grafana` in a Docker container (useful when the binary is not installed locally):

```json
{
  "mcpServers": {
    "grafana": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "GRAFANA_URL",
        "-e", "GRAFANA_SERVICE_ACCOUNT_TOKEN",
        "ghcr.io/grafana/mcp-grafana:latest"
      ],
      "env": {
        "GRAFANA_URL": "http://host.docker.internal:3000",
        "GRAFANA_SERVICE_ACCOUNT_TOKEN": "<your-service-account-token>"
      }
    }
  }
}
```

Use `host.docker.internal` to reach a Grafana instance running on the host machine from within Docker.

### Using Claude with VictoriaMetrics dashboards

Once configured, you can ask Claude to interact with your VictoriaMetrics data through Grafana, for example:

* _"List all dashboards that use the VictoriaMetrics datasource."_
* _"Show me the current ingestion rate from the VictoriaMetrics overview dashboard."_
* _"Run a MetricsQL query `rate(vm_rows_inserted_total[5m])` against the VictoriaMetrics datasource."_

See [mcp-grafana documentation](https://github.com/grafana/mcp-grafana) for the full list of available tools.
