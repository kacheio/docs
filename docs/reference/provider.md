# Provider

A provider is responsible for the actual storage of HTTP responses. A provider implementation can satisfy different requirements such as persistence, performance, and distribution, from local RAM caches to globally distributed persistent caches. They can be fully custom caches or wrappers and adapters for local or remote open-source or proprietary caches. 

At the moment, the available and supported cache storage implementations are In-memory and Redis.

* [In-memory](#in-memory-cache)
* [Remote](#remote-cache)


### In-Memory cache

In-memory stores HTTP responses in a local in-memory cache. It is a volatile storage and does not persist the entries to disk. It is also not shared between multiple Kache instances. The in-memory cache is size-bound and ensures the total cache size approximately does not exceed the configured `max_size` in bytes. The following shows an example configuration of an in-memory cache with maximum overall cache size and maximum single item size constraints and with TTL eviction enabled and set to 120s.

=== "YAML"
    ```yaml
    provider:
        # Activate inmemory as cache backend.
        backend: inmemory
        
        # Configure inmemory cache.
        inmemory:
            # Overall cache size of 1GB.
            max_size: 1000000000
            # Max item size of 50MB.
            max_item_size: 50000000
            # Items expire after 120s.
            default_ttl: 120s
    ```

!!! tip
    To disable TTL eviction, set `default_ttl` to -1.

### Remote cache

A remote cache is a distributed remote cache such as Redis or Memcached. The following example configures Redis as a remote caching provider.

=== "YAML"
    ```yaml
    provider:
        # Activate redis as cache backend.
        backend: redis

        # Configure redis remote cache.
        redis:
            endpoint: "redis:6379" 
            username:
            password:
            db: 
            max_item_size: 50000000
            max_queue_concurrency: 56
            max_queue_buffer_size: 24000

    ```

### Layered cache

A layered or tiered cache adds a local in-memory caching layer on top of a remote cache provider (typically a distributed and shared network cache). Items will always be stored in both caches. Fetches are only satified by the underlying remote cache, if the item does not exist in the local cache. The local cache will remove items, depending on the capacity constraints of the cache or the lifetime constraints of the cached item, respectively. This strategy can be used to speed up operations when working with remote network caches.

The following example configures a remote cache that is layered by a local in-memory cache.

=== "YAML"
    ```yaml
    provider:
        # Activate redis as the main remote cache backend.
        backend: redis

        # Activate layered caching strategy.
        layered: true

        // Configure remote redis cache (layer 2).
        redis:
            endpoint: "redis:6379"
            username:
            password:
            db:
            max_item_size: 50000000
            max_queue_concurrency: 56
            max_queue_buffer_size: 24000

        // Configure local inmemory cache (layer 1).
        inmemory:
            max_size: 1000000000
            max_item_size: 50000000
            default_ttl: 120s
    ```

### Reference

##### General

| Directive     | Type        | Description                          |
| -----------   | ----------- | ------------------------------------ |
| `backend`     | `string`    | Specifies the backend used as the underlying cache storage provider. Valid values are `inmemory` for a local in-memory cache and `redis` for the distributed remote cache based on Redis.  |
| `layered`     | `bool`      | Activates a two-tier cache strategy, putting a local cache layer on top of the remote cache to speed up operations. |

##### In-Memory cache

| Directive       | Type        | Description                          |
| -----------     | ----------- | ------------------------------------ |
| `max_size`      | `int`       | Sets the overall maximum number of bytes the cache can hold.  |
| `max_item_size` | `int`       | Sets the maximum size of a single item. |
| `default_ttl`   | `string`    | Sets the defautl TTL of a single item. The value is a duration string. A duration string is a possibly signed sequence of decimal numbers, each with optional fraction and a unit suffix, such as "300ms", "1.5h", or "2h45m". Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h". A valid TTL must be greater than 0. If TTL is set to -1, TTL eviction is disabled. |

##### Redis remote cache

| Directive                 | Type        | Description                          |
| -----------               | ----------- | ------------------------------------ |
| `endpoint`                | `string`    | Specifies the endpoint addresses of the Redis server. Either a single address or a comma-separated list of host:port addresses of cluster/sentinel nodes. |
| `username`                | `string`    | Sets the username to authenticate the current connection with one of the connections defined in the ACL list when connecting to a Redis 6.0 instance, or greater, that is using the Redis ACL system. |
| `password`                | `string`    | Optional password. Must match the password specified in the requirepass server configuration option (if connecting to a Redis 5.0 instance, or lower), or the User Password when connecting to a Redis 6.0 instance, or greater, that is using the Redis ACL system. |
| `db`                      | `int`       | Sets the database to be selected after connecting to the server. Default is `0`. |
| `max_item_size`           | `int`       | Specifies the maximum size of an item stored in Redis. Items bigger than MaxItemSize are skipped. If set to 0, no maximum size is enforced. |
| `max_queue_concurrency`   | `int`       | Specifies the maximum number of enqueued job operations allowed. |
| `max_queue_buffer_size`   | `int`       | Specifies the maximum number of concurrent asynchronous job operations. |

