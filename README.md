# Docker Stuff

## Info

```sh
$ docker version
$ docker info
$ docker events
```

## Images

### List

```sh
$ docker images
```

### Pull New

```sh
$ docker pull centos
$ docker pull -a centos

$ for x in busybox centos scratch registry hipache debian ubuntu-upstart nginx node mysql postgres redis java golang swarm logstash rails kibana ruby gcc haskell mongo nats pypy mono couchbase jruby percona thrift cassandra; do; docker pull $x; done
```

### Remove

```sh
$ docker rmi centos
```

### Visualize the tree of images

Install [dockviz](https://github.com/justone/dockviz)

```sh
$ dockviz images --tree
$ alias d-tree='dockviz images -t'
$ alias d-graph='dockviz images -d | dot -Tpng -o /tmp/dockviz && open /tmp/dockviz'
$ alias dc-graph='dockviz containers -d | dot -Tpng -o /tmp/dockviz && open /tmp/dockviz'
```

### Packaging an image to a tar

```sh
$ docker save -o /tmp/cool-file-image.tar cool_file_image

# you can see the layers by
$ tar -tf /tmp/cool-file-image.tar
```

### Loading image from file

```sh
$ docker load -i /tmp/cool-file-image.tar
```

## Containers

### Run in Foreground (interactive)

```sh
$ docker run -it --name shell alpine /bin/sh
$ docker run -it --name pinger alpine /bin/sh -c "ping 8.8.8.8"

# -i for interactive
# -t for tty

# to detach, do: ctrl+p, ctrl+q
```

### Run in Background (detached)

```sh
$ docker run -d --name pinger alpine /bin/sh -c "ping 8.8.8.8"

# -d for detached
```

### List Containers

```sh
$ docker ps # running containers
$ docker ps -a # include terminated
```

### Stopping

```sh
$ docker stop pinger
$ docker kill pinger
$ docker kill -s SIGKILL pinger
```

### Starting

```sh
$ docker start pinger
```

### Pausing

```sh
$ docker pause pinger
$ docker unpause pinger
```

### Attaching

```sh
$ docker attach pinger

# to detach, do: ctrl+p, ctrl+q
```

### Inspecting

```sh
$ docker inspect pinger
$ docker inspect pinger | grep -i PID
```

### Live statistics

```sh
$ docker stats pinger
```

### Top

```sh
$ docker top pinger
```

### Logs

```sh
$ docker logs -f pinger
```

### Diff

```sh
$ docker diff pinger
```

### Debug

Run a new command in an already running container

```sh
$ docker exec -it pinger /bin/sh
```

If you have nsenter on host, you can also do

```sh
$ nsenter -m -u -n -p -i -t 19867 /bin/bash

# where 19867 is the process id of the guest process in the host system, ref: inspect
```

### Removing Container

```sh
$ docker rm -f pinger
```

### Saving changes made to a container into an image

Make change to file system

```sh
$ docker run --name cool_file_container alpine /bin/sh -c "echo 'cool content' > /tmp/cool-file"
```

Save container

```sh
$ docker commit cool_file_container cool_file_image
```

### History

```sh
$ docker history --no-trunc=true cool_file_image
```
