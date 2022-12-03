# Description

[Fluent Bit](https://fluentbit.io) input plugin able to access [Docker Container Inspect](https://docs.docker.com/engine/api/v1.41/#operation/ContainerInspect)

# Requirements

- Docker
- Docker image `fluent-beats/fluent-bit-plugin-dev`

# Build
```bash
./build.sh
```

# Test
```bash
./test.sh
 ```

 # Design

  ## Required volumes

 `Fluentbit` provides very simple libraries to deal with JSON processing, so to make this plugin bare simple it uses a simple strategy to collect info about all container running in the `host node`.

 First it maps the folder `/var/lib/docker/containers` so it can simply know all containers' ids running in the underline host without any request/response to Docker Engine API.

 Second it maps the **host unix socket** `/var/run/docker.sock`, so it can send requests to the Docker Engine API to fetch stats from all containers retrieved previously from mounted `/var/lib/docker/containers`.

> This plugin WILL NOT WORK without both volumes properly mounted on `Fluentbit` container


## Collecting info
The plugin will access the Docker Engine endpoint `/containers/{id}/json`.

It will use the following parameters:
 * size=true (return information about container size)

Each response will include important information like:

 - Container health / state
 - Container image
 - Container labels
 - Container volumes and ports

 For each response the plugin will simply send the JSON values to the output without any transformation or parsing, saving even more memory and resources.

 ## Configurations

This input plugin can be configured using the following parameters:

| Key  | Description | Default |
| ---- | ----------- | ------ |
| Collect_Interval  | Interval in seconds to collect data  | 10 |
| Unix_Path | Define target Unix socket path. | /var/run/docker.sock
| Containers_Path | Define the folder that contains containers' ids. | /var/lib/docker/containers |
| Buffer_Size | The size of the buffer used to read data (in bytes or [unit sized](https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/unit-sizes))  | 12288 |
| Parser | Specify the name of a parser to interpret the entry as a structured message. | None |
| Key | When a message is unstructured (no parser applied), it's appended as a string under the key name message. | message |