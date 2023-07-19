# HTTP Cache

The HTTP cache acts as a filter or middleware sitting between listeners and upstream targets. All incoming requests are routed through the HTTP cache, which implements most of the complexities of HTTP caching semantics and relevant RFC specifications[^1].

For HTTP Requests:

* Considers the Cache-Control header of the request. For example, if the request has `Cache-Control: no-store`, the request is not cached.
* Does not store HTTP HEAD requests.

For HTTP Responses:

* Only caches responses with enough data to calculate the freshness lifetime[^2].
* Considers the Cache-Control header from the upstream host. For example, if the HTTP response returns with status code `200` and `Cache-Control: max-age=60`, the response will be cached.
* Only caches responses with following status codes:`200, 203, 204, 206, 300, 301, 308, 404, 405, 410, 414, 451, 501`.

The actual storage of HTTP responses is delegated to the implementations of a caching provider. These implementations can cover different requirements such as persistence, performance, and distribution, from local RAM caches to globally distributed persistent caches. They can be fully custom caches or wrappers and adapters for local or remote open source or proprietary caches. Currently, the available cache storage implementations are In-memory and Redis. More informations and relevant configurations of caching providers can be foud [here](./provider.md).

## Configuration

Kache provides several options for customizing the caching behavior. These configurations can be changed either permanently via the configuration file or temporarily via the configuration API. See the [Configuration reference](../intro/configuration.md) and [API reference](./api_specification.md#cache-configuration) for detailed explanations.

### Custom headers

Kache adds custom headers (X-Header) to each response that is served by Kache. These headers contain information about whether the response was delivered from the cache or not, and can be useful for debugging. 

The headers are disabled by default and must be enabled in the configuration. Once enabled, debug headers are added to each response and contain information about whether the response was delivered from the cache or not. 

They are presented in a canonical format, where `x_header_name` specifies the name of the corresponding header entry and its value indicating whether it is a cache `HIT` or `MISS`. If no header name is specified in the configuration, the default header name (`X-Kache`) is used.

=== "YAML"
  ```yaml
  cache:
    # Activate debug header.
    x_header: true
    # Set debug header name to 'X-Kache'.
    x_header_name: x-kache
  ```


### Cache-Control

The HTTP header field Cache-Control contains directives (instructions) – both in requests and responses – that control caching in browsers and shared caches (e.g. proxies). The Cache-Control header specifies directives that must be followed by all caching mechanisms. 

Kache fully implements the latest RFC specification[^1] and respects these directives when caching responses to requests. However, in some cases it is desirable to customize directives sent by backends, or to add directives for responses that do not include a Cache-Control header at all. Therefore, Kache allows setting or modifying the Cache-Control header field of such upstream responses. It is, however, advisable to not change the Cache-Control values set by backend servers and to do so only when necessary, as this could lead to undesirable side effects.

If `default_cache_control_header` is specified in the configuration, a Cache-Control header with the corresponding value will be added to any response that does not contain a Cache-Control header. If the backend sets a header, overriding it with the configured default header can be enforced via the `force_cache_control` directive.

For example, the following configuration sets a custom Cache-Control header field with a `max-age` of 2 minutes (or 120 seconds) on each incoming upstream response and defines for how long the resource is cachable. Note that `max-age` is a relative time defined in seconds. If the upstream response already contains a Cache-Control header, it will be overwritten due to `force_cache_control` set to `true`.

=== "YAML"
  ```yaml
  cache:
    # Set default cache-control header if missing or enforced to update.
    default_cache_control: "max-age=120"
    # Always set the specified default cache-control regardless if present or not.
    force_cache_control: true
  ```

### Expiration

The lifetime of a cached item is represented by its TTL (time-to-live). An item lives in the cache until its TTL elapses and the cached resource expires. After that time, the item is removed from the cache. Resources within their TTL are considered fresh, while stale resources are those whose lifetime has already expired. Setting the right lifetime durations is fundamental for a healthy cache, avoids consuming valuable system resources such as cache memory, and ensures that users have an optimal experience.

For a HTTP cache that stores responses, the TTL can be set via [HTTP header](#cache-control) either by the origin servers or by Kache itself. If set by the origin server, Kache respects those values. Changing or overriding those values is still possible through the configuration. If no TTLs have been set by the origin server, Kache applies default and custom TTLs that can be defined per path and resource. 

For example, the following configuration configures the HTTP cache with a default TTL (time-to-live) of 1200s for each resource, but applies custom and more fine-grained TTLs to specific paths and resources.

=== "YAML"
  ```yaml
  cache:
    # Default TTL in seconds.
    default_ttl: 1200s

    # Custom TTLs per path/resouce.
    timeouts:
      - path: "/news"
        ttl: "10s"
      - path: "/archive"
        ttl: "86400s"
      - path: "^/assets/([a-z0-9].*).css"
        ttl: "120s"
  ```      

### Ignore

Any resource is cached and stored as long as it is [cacheable](#http-cache) and fresh (has not expired).

To explicitly ignore certain requests and responses, Kache provides the ability to exclude those resources from being cached. This can be done either for __requests__ with a specific path, for __requests__ with a specific header, or for __responses__ containing objects of a specific content type and size.

For example, the following configuration specifies that requests to `/admin` are bypassed by Kache and thus excluded from caching. The same is true for requests containing a request header with key `X-Requested-With` and value `XMLHttpRequest`. In addition, responses containing javascript, css, and images with a size greater than 1MB are not cached as well. 

=== "YAML"
  ```yaml
  cache:
    # Exclude resources from cache.
    exclude:
        # Exclude all requests matching the specified path (regex).
        path:
          - "^/admin" 
          - "^/.well-known/acme-challenge/(.*)"
        # Exclude all request with a specific header field and value.
        header:
          x_requested_with: "XMLHttpRequest"
        # Don't cache responses depending on their type and size.
        content: # applied to responses
          - type: "text/javascript|text/css|image/.*"
            size: 1000000 # in bytes
  ```

## Reference

| Directive                 | Type        | Description |
|-------------------------- |------------ |------------ |
| `x_header`                | `bool`      | Activates the X-Cache debug header. If set to `true`, Kache will add a HTTP response header to each response indicating if it was served from cache or not (cache hit or miss). |
| `x_header_name`           | `string`    | Specifies the name of the X-Cache debug header. For example, if set to `x-kache` and in case of a cache hit, the response will contain an additional HTTP header with `X-Kache` as key and `HIT` as value. Default is 'X-Kache'. |
| `default_ttl`             | `string`    | Is the default TTL (time-to-live) for cached items. The value is a duration string. A duration string is a possibly signed sequence of decimal numbers, each with optional fraction and a unit suffix, such as "300ms", "1.5h", or "2h45m". Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h". TTL must be greater than 0. |
| `default_cache_control`   | `string`    | Specifies a custom Cache-Control header. If configured, the custom Cache-Control header is added to each response that does not contain a valid Cache-Control header. The value must be a valid directive according to the corresponding [RFC specification](https://httpwg.org/specs/rfc7234#header.cache-control).  |
| `force_cache_control`     | `bool`      | Specifies whether to overwrite and modify an existing Cache-Control header. If set to `true`, the Cache-Control header set by the origin server will be overwritten with the value set in `default_cache_control`. |
| `timeouts`                | `list`      | Configuration section for custom timeouts (list of TTLs per path). |
| `path`                    | `string`    | The path to the resource the TTL is applied to. Accepts a string or a valid regex. |
| `ttl`                     | `string`    | The duration after the specified resource expires. Accepts a valid duration string. |
| `excluded`                | `list`      | Configuration section for resources excluded from cache. |
| `path`                    | `string`    | The request path that will be ignored. Accepts a string or a valid regex. |
| `header`                  | `string`    | Headers to be ignored by the cache. Header names must be specified in [snake case](https://en.wikipedia.org/wiki/Snake_case) format which is then resolved to a canonical format. E.g. to match the canonical header name X-Requested-With, the corresponding value in the configuration must be `x_requested_with`. |
| `content`                 | `list`      | Configuration section containing the content ignored by the cache. List of type and size (size is optional). |
| `type`                    | `string`    | Type is the content type to be ignored by the cache. Accepts a string or a valid regex. See [here](https://www.iana.org/assignments/media-types/media-types.xhtml) for an up-to-date and complete list of media types. |
| `size`                    | `int`       | Size is the max content size in bytes. For the every corresponding matching content type in `type`, resources that exceed the specified size are not cached. |

[^1]:
    [RFC7234 -- HTTP Caching](https://httpwg.org/specs/rfc7234)
    

[^2]:
    [RFC7234 -- Calculating freshness lifetime](https://httpwg.org/specs/rfc7234.html#calculating.freshness.lifetime)