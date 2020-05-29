# GoCD Agent Docker image

[GoCD agent](https://www.gocd.org) docker image based on ubuntu 16.04.


# Issues, feedback?

Please make sure to log them at https://github.com/gocd/gocd.

# Usage

Start the container with this:

```
docker run -d -e GO_SERVER_URL=... gocd/gocd-agent-ubuntu-16.04:v20.4.0
```

**Note:** Please make sure to *always* provide the version. We do not publish the `latest` tag. And we don't intend to.

This will start the GoCD agent and connect it the GoCD server specified by `GO_SERVER_URL`.

> **Note**: The `GO_SERVER_URL` must be an HTTPS url and end with `/go`, for e.g. `http://ip.add.re.ss:8153/go`

## Usage with docker GoCD server

If you have a [gocd-server container](https://hub.docker.com/r/gocd/gocd-server/) running and it's named `angry_feynman`, you can connect a gocd-agent container to it by doing:

```
docker run -d -e GO_SERVER_URL=http://$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' angry_feynman):8153/go gocd/gocd-agent-ubuntu-16.04:v20.4.0
```
OR

If the docker container running the gocd server has ports mapped to the host,

```
docker run -d -e GO_SERVER_URL=http://<ip_of_host_machine>:$(docker inspect --format='{{(index (index .NetworkSettings.Ports "8153/tcp") 0).HostPort}}' angry_feynman)/go gocd/gocd-agent-ubuntu-16.04:v20.4.0
```

# Available configuration options

## Auto-registering the agents

```
docker run -d \
        -e AGENT_AUTO_REGISTER_KEY=... \
        -e AGENT_AUTO_REGISTER_RESOURCES=... \
        -e AGENT_AUTO_REGISTER_ENVIRONMENTS=... \
        -e AGENT_AUTO_REGISTER_HOSTNAME=... \
        gocd/gocd-agent-ubuntu-16.04:v20.4.0
```

If the `AGENT_AUTO_REGISTER_*` variables are provided (we recommend that you do), then the agent will be automatically approved by the server. See the [auto registration docs](https://docs.gocd.org/20.4.0/advanced_usage/agent_auto_register.html) on the GoCD website.

## Configuring SSL

To configure SSL parameters, pass the parameters using the environment variable `AGENT_BOOTSTRAPPER_ARGS`. See [this documentation](https://docs.gocd.org/20.4.0/installation/ssl_tls/end_to_end_transport_security.html) for supported options.

```shell
    docker run -d \
    -e AGENT_BOOTSTRAPPER_ARGS='-sslVerificationMode NONE ...' \
    gocd/gocd-agent-ubuntu-16.04:v20.4.0
```

## Usage with docker and swarm elastic agent plugins

This image will work well with the [docker elastic agent plugin](https://github.com/gocd-contrib/docker-elastic-agents) and the [docker swarm elastic agent plugin](https://github.com/gocd-contrib/docker-swarm-elastic-agents). No special configuration would be needed.
## Mounting volumes

The GoCD agent will store all configuration, logs and perform builds in `/godata`. If you'd like to provide secure credentials like SSH private keys among other things, you can mount `/home/go`.

```
docker run -v /path/to/godata:/godata -v /path/to/home-dir:/home/go gocd/gocd-agent-ubuntu-16.04:v20.4.0
```

> **Note:** Ensure that `/path/to/home-dir` and `/path/to/godata` is accessible by the `go` user in container (`go` user - uid 1000).

## Tweaking JVM options (memory, heap etc)

JVM options can be tweaked using the environment variable `GOCD_AGENT_JVM_OPTS`.

```
docker run -e GOCD_AGENT_JVM_OPTS="-Dfoo=bar" gocd/gocd-agent-ubuntu-16.04:v20.4.0
```

# Under the hood

The GoCD server runs as the `go` user, the location of the various directories is:

| Directory           | Description                                                                      |
|---------------------|----------------------------------------------------------------------------------|
| `/godata/config`    | the directory where the GoCD configuration is store                              |
| `/godata/pipelines` | the directory where the agent will run builds                                    |
| `/godata/logs`      | the directory where GoCD logs will be written out to                             |
| `/home/go`          | the home directory for the GoCD server                                           |

## Running docker and docker-compose in your jobs

To be able to run the `docker` and `docker-compose` commands inside your jobs, you will need to share the docker socket as a volume from your host which is pretty classic.

In this case, as the docker deamon will be the one mounting the volumes you define, the path to the files you will want to mount (basically inside `/godata/pipelines`) need to be the same so that the docker deamon (which is running on the host) can find the files.

If you run several agents container, you will need to overwrite the `VOLUME_DIR` environment variable to have a different path for your `/godata` for each of your gocd agent containers (to avoid issues). For example, if the volume on your host for the first container is `/go-agent1/godata`, you will set the `VOLUME_DIR` environment data on your container to `/go-agent1/godata` and the `docker-entrypoint.sh` script will automatically manage it and make sure the agent stores its configuration, logs and pipelines there.

# Running GoCD Containers as Non Root

With release `v19.6.0`, GoCD containers will run as non-root user, by default. The Dockerized GoCD application will run with user `go` (uid: `1000`) and group `root` (gid: `0`) instead of running as user `root` (uid: `0`) and group `root` (gid: `0`). For more information, checkout [Running Dockerized GoCD Containers as Non Root](https://www.gocd.org/2019/06/25/GoCD-non-root-containers/) blog post.

# Troubleshooting

## The GoCD agent does not connect to the server

- Check if the docker container is running `docker ps -a`
- Check the STDOUT to see if there is any output that indicates failures `docker logs CONTAINER_ID`
- Check the agent logs `docker exec -it CONTAINER_ID /bin/bash`, then run `less /godata/logs/*.log` inside the container.

# License

```plain
Copyright 2020 ThoughtWorks, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
