# Installation


You can install and run Kache in the following ways:

* [Use the official Docker image](#docker)
* [Use the binary distribution](#use-the-binary-distribution)
* [Build binary from source](#use-the-binary-distribution)
* [Use the Helm Chart](#use-the-helm-chart)


### <a name="docker"></a> Use the Official Docker Image

Use one of the official [Docker images](https://hub.docker.com/r/kacheio/kache) and run it with the sample configuration file:

```
docker run -d -p 8080:8080 -p 80:80 \
    -v $PWD/kache.yml:/etc/kache/kache.yml \
    kache:latest -config.file=/etc/kache/kache.yml 
```

### Use the binary distribution

To run kache, get the latest binary from the [releases](https://github.com/kacheio/kache/releases) page and run it with the [sample configuration file](https://github.com/kacheio/kache/blob/main/kache.sample.yml):

```
./kache -config.file=kache.yml
```

### Build binary from source

```
git clone https://github.com/kacheio/kache
```

### Use the Helm Chart

!!! Info
    Comming soon!


### Quick Start

If you want to run kache with a distributed caching backend (e.g. Redis), you can use and run this example [docker-compose](https://github.com/kacheio/kache/blob/main/cloud/docker-compose.yml) as a starting point:

```
docker-compose -f cloud/docker-compose.yml up 
```

!!! tip
    Check the Quick Starts for [Docker](./quick-start.md) and [Kubernetes](./quick-start-k8s.md) to learn more.
