# Docker Stuff

## Images

List images

```sh
$ docker images
```

Pull a new image

```sh
$ docker pull centos
$ for x in busybox centos scratch registry hipache debian ubuntu-upstart nginx node mysql postgres redis java golang swarm logstash rails kibana ruby gcc haskell mongo nats pypy mono couchbase jruby percona thrift cassandra; do; docker pull $x; done
```
