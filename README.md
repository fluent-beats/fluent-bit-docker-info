# Description

[Fluent Bit](https://fluentbit.io) input plugin able to access [Docker Container Info](https://docs.docker.com/engine/api/v1.41/#operation/ContainerList)

# Requirements

- Docker
- Docker image `fluent-beats/fluent-bit-plugin-dev`

# Build
```bash
./build.sh
```

# Test
```bash
docker run --rm \
      -v /var/run/docker.sock:/var/run/docker.sock:ro \
      -v /var/lib/docker/containers:/var/lib/docker/containers \
      -v $(pwd)/code/build:/my_plugin \
      fluent/fluent-bit:1.8.4 /fluent-bit/bin/fluent-bit -e /my_plugin/flb-in_docker_info.so -i docker_info -o stdout
 ```

 # Design

 ## Required volumes

 ## Required volumes

 `Fluentbit` provides very simple libraries to deal with JSON processing, so to make this plugin bare simple it uses a simple strategy to collect info about all container running in the `host node`.

 First it maps the folder `/var/lib/docker/containers` so it can simply know all containers' ids running in the underline host without any request/response to Docker Engine API.

 Second it maps the **host unix socket** `/var/run/docker.sock`, so it can send requests to the Docker Engine API to fetch stats from all containers retrieved previously from mounted `/var/lib/docker/containers`.

> This plugin WILL NOT WORK without both volumes properly mounted on `Fluentbit` container


## Collecting info
The plugin will access the Docker Engine endpoint `/containers/json`.

It will use the following parameters:
 * all=true (return information even if container is not running)
 * filters={"id":$id} (limits the response to a single container)

Each response will include important information like:

 - Container health / state
 - Container image
 - Container labels
 - Container volumes and ports

 For each response the plugin will simply send the JSON values to the output without any transformation or parsing, saving even more memory and resources.