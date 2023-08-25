# File

=== "YAML"
  ``` yaml
  listeners:
    web1:
      addr: :80
    web2:
      addr: :3128

  upstreams:
    - name: service1
      addr: "http://localhost:8000"
      path: "/service/1"

    - name: service2
      addr: "http://example.com"

  api:
    port: 6067
    debug: true

  logging:
    level: debug # trace, debug, info, warn, error, fatal, panic
    color: true

  cache:
    x_header: true
    x_header_name: x-kache
    default_ttl: 1200s

  provider:
    backend: redis
    layered: true

    redis:
      endpoint: "redis:6379"
      username:
      password:
      db:
      max_item_size: 10000000
      max_queue_concurrency: 56
      max_queue_buffer_size: 24000

    inmemory:
      max_size: 1000000000 
      max_item_size: 50000000
      default_ttl: 120s

  ```