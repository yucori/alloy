---
canonical: https://grafana.com/docs/alloy/latest/reference/components/discovery/discovery.file/
aliases:
  - ../discovery.file/ # /docs/alloy/latest/reference/components/discovery.file/
description: Learn about discovery.file
labels:
  stage: general-availability
  products:
    - oss
title: discovery.file
---

# `discovery.file`

`discovery.file` discovers targets from a set of files, similar to the [Prometheus `file_sd_config`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config).

## Usage

```alloy
discovery.file "<LABEL>" {
  files = ["<FILE_PATH_1>", "<FILE_PATH_2>", ...]
}
```

## Arguments

You can use the following arguments with `discovery.file`:

| Name               | Type           | Description                              | Default | Required |
| ------------------ | -------------- | ---------------------------------------- | ------- | -------- |
| `files`            | `list(string)` | Files to read and discover targets from. |         | yes      |
| `refresh_interval` | `duration`     | How often to sync targets.               | `"5m"`  | no       |

The last path segment of each element in `files` may contain a single `*` that matches any character sequence, for example, `my/path/tg_*.json`.

## Blocks

The `discovery.file` component doesn't support any blocks. You can configure this component with arguments.

## Exported fields

The following fields are exported and can be referenced by other components:

| Name      | Type                | Description                                        |
| --------- | ------------------- | -------------------------------------------------- |
| `targets` | `list(map(string))` | The set of targets discovered from the filesystem. |

Each target includes the following labels:

* `__meta_filepath`: The absolute path to the file the target was discovered from.

## Component health

`discovery.file` is only reported as unhealthy when given an invalid configuration.
In those cases, exported fields retain their last healthy values.

## Debug information

`discovery.file` doesn't expose any component-specific debug information.

## Debug metrics

`discovery.file` doesn't expose any component-specific debug metrics.

## Examples

The following examples show you how to configure the `discovery.file` component.

### Example target files

```json
[
  {
    "targets": [ "127.0.0.1:9091", "127.0.0.1:9092" ],
    "labels": {
      "environment": "dev"
    }
  },
  {
    "targets": [ "127.0.0.1:9093" ],
    "labels": {
      "environment": "prod"
    }
  }
]
```

```yaml
- targets:
  - 127.0.0.1:9999
  - 127.0.0.1:10101
  labels:
    job: worker
- targets:
  - 127.0.0.1:9090
  labels:
    job: prometheus
```

### Basic file discovery

This example discovers targets from a single file, scrapes them, and writes metrics to a Prometheus remote write endpoint.

```alloy
discovery.file "example" {
  files = ["/tmp/example.json"]
}

prometheus.scrape "default" {
  targets    = discovery.file.example.targets
  forward_to = [prometheus.remote_write.demo.receiver]
}

prometheus.remote_write "demo" {
  endpoint {
    url = "<PROMETHEUS_REMOTE_WRITE_URL>"

    basic_auth {
      username = "<USERNAME>"
      password = "<PASSWORD>"
    }
  }
}
```

Replace the following:

* _`<PROMETHEUS_REMOTE_WRITE_URL>`_: The URL of the Prometheus remote_write-compatible server to send metrics to.
* _`<USERNAME>`_: The username to use for authentication to the `remote_write` API.
* _`<PASSWORD>`_: The password to use for authentication to the `remote_write` API.

### File discovery with retained file path label

This example discovers targets from a wildcard file path, scrapes them, and writes metrics to a Prometheus remote write endpoint.

It also uses a relabeling rule to retain the file path as a label on each target.

```alloy
discovery.file "example" {
  files = ["/tmp/example_*.yaml"]
}

discovery.relabel "keep_filepath" {
  targets = discovery.file.example.targets
  rule {
    source_labels = ["__meta_filepath"]
    target_label = "filepath"
  }
}

prometheus.scrape "default" {
  targets    = discovery.relabel.keep_filepath.output
  forward_to = [prometheus.remote_write.demo.receiver]
}

prometheus.remote_write "demo" {
  endpoint {
    url = <PROMETHEUS_REMOTE_WRITE_URL>

    basic_auth {
      username = <USERNAME>
      password = <PASSWORD>
    }
  }
}
```

Replace the following:

* _`<PROMETHEUS_REMOTE_WRITE_URL>`_: The URL of the Prometheus remote_write-compatible server to send metrics to.
* _`<USERNAME>`_: The username to use for authentication to the `remote_write` API.
* _`<PASSWORD>`_: The password to use for authentication to the `remote_write` API.

<!-- START GENERATED COMPATIBLE COMPONENTS -->

## Compatible components

`discovery.file` has exports that can be consumed by the following components:

- Components that consume [Targets](../../../compatibility/#targets-consumers)

{{< admonition type="note" >}}
Connecting some components may not be sensible or components may require further configuration to make the connection work correctly.
Refer to the linked documentation for more details.
{{< /admonition >}}

<!-- END GENERATED COMPATIBLE COMPONENTS -->
