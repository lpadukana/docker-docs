# Docker Stuff

## Docker machines

For boot2docker

```sh
$ docker-machine create -d virtualbox dev
```

For RancherOS

```sh
$ docker-machine create -d virtualbox --virtualbox-boot2docker-url https://github.com/rancher/os/releases/download/v0.3.3/machine-rancheros.iso ros
```

## Info

```sh
$ docker version
$ docker info
$ docker events
```

## Shell access to host

```sh
$ docker-machine ssh dev
```

```sh
$ docker-machine scp /etc/hosts dev:/tmp/
```

## Port NATing to localhost

```sh
# PURELY COSMETIC: enable access to common ports on the docker host VM
# this will nat the ports from $(docker-machine ip dev) to 'localhost'

$ docker-machine stop dev

$ VBoxManage modifyvm "dev" --natpf1 "tcp-port3000,tcp,,3000,,3000"
$ VBoxManage modifyvm "dev" --natpf1 "tcp-port4000,tcp,,4000,,4000"
$ VBoxManage modifyvm "dev" --natpf1 "tcp-port8000,tcp,,8000,,8000"
$ VBoxManage modifyvm "dev" --natpf1 "tcp-port8001,tcp,,8001,,8001"
$ VBoxManage modifyvm "dev" --natpf1 "tcp-port8080,tcp,,8080,,8080"
$ VBoxManage modifyvm "dev" --natpf1 "tcp-port9000,tcp,,9000,,9000"
$ VBoxManage modifyvm "dev" --natpf1 "tcp-port9001,tcp,,9001,,9001"

$ docker-machine start dev

eval "$(docker-machine env dev)"
```

## Images

### List

```sh
$ docker images
```

### Pull New

```sh
$ docker pull centos
```
Other interesting images include busybox debian ubuntu-upstart nginx node mysql redis java golang rails ruby jruby gcc haskell mongo nats pypy mono couchbase postgres cassandra swarm logstash kibana

Note: If you see something like "Network timed out while trying to connect to https://index.docker.io/v1/repositories/library/mysql/images.", you might want to restart your vm!

### Remove

```sh
$ docker rmi centos

# remove all images
$ docker rmi $(docker images -qf "dangling=true")
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
$ docker run -it --rm --name shell alpine /bin/sh
$ docker run -it --rm --name pinger alpine /bin/sh -c "ping 8.8.8.8"

# --rm for automatically removing container on exit
# -i for interactive
# -t for tty

# to detach, do: ctrl+p, ctrl+q
```

### Run in Background (detached)

```sh
$ docker run -d --name pinger alpine /bin/sh -c "ping 8.8.8.8"
$ docker run -d --name pinger --restart=always alpine ping -c4 8.8.8.8

# -d for detached
# --restart for restart policy (no, always, on-failure, on-failure:10)
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

### Ports

```sh
$ docker run --name web1 -d -P nginx
$ docker run --name web2 -d -p 8080:80 nginx

$ docker port web1
$ docker port web1 80

$ curl "$(docker-machine ip dev):$(docker port web1 80 | cut -d':' -f2)"
$ curl "$(docker-machine ip dev):$(docker port web2 80 | cut -d':' -f2)"
```

### Volumes

```sh
$ mkdir site && cd site
$ echo 'Man, docker volumes!' > index.html
$ docker run --name web -d -P -v $PWD:/usr/share/nginx/html nginx
# To mount in readonly mode, $ docker run --name web -d -P -v $PWD:/usr/share/nginx/html:ro nginx

$ curl "$(docker-machine ip dev):$(docker port web 80 | cut -d':' -f2)"
# you should see: "Man, docker volumes!"

$ echo 'Another cool page!' > cool.html
$ curl "$(docker-machine ip dev):$(docker port web 80 | cut -d':' -f2)/cool.html"
# you should see: "Another cool page!"

$ cd .. && rm -rf site
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

# remove all stopped containers
$ docker rm $(docker ps -q -f status=exited)

# remove all stopped containers, force stop if needed
$ docker rm -f `docker ps --no-trunc -aq`

$ docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs --no-run-if-empty docker rm
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
