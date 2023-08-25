# Quick Start

The following describes how to run kache locally with Docker.

### Set up a docker-compose

Create and define a [docker-compose.yml](https://github.com/kacheio/kache/blob/main/deploy/docker/docker-compose.yml) and define a kache service that uses the official [Kache image](https://hub.docker.com/r/kacheio/kache):

``` yaml
services:

  kache:
    # Use the `main` tag for the latest development image 
    # or `latest` tag for the latest stable version.
    image: kacheio/kache:main
    container_name: kache
    # Start the container with the mounted config file.
    command:
      - "-config.file=/etc/kache/kache.sample.yml"
    # Expose ports that are configured in the kache config file.
    ports:
      - "80:80"
      - "3129:3129"
      - "6067:6067"
    # Mount the config file.
    volumes:
      - "./../kache.sample.yml:/etc/kache/kache.sample.yml"

  # Use redis as distributed remote caching backend.
  redis:
    image: "redis:alpine"

```

That's all you need to run Kache with Redis as a distributed caching backend. 


### Run kache

Now start Kache with the following command:

```
docker-compose -f deploy/docker/docker-compose.yml up 
```

### Access kache
You can now access the service under `http://localhost:3129/` and the API under `http://localhost:6067/api/cache/keys` which, for example, returns all the keys currently in the cache.