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

### Configuration

The following example configures the HTTP cache with a default TTL (time-to-live) of 1200s and activates a custom debug header (X-Header) attached to every response that is served from Kache. Debug headers are added to the request and contain information if the response was served from cache or not. They are presented in a canonical format where `x_header_name` specifies the name of the header entry with its value indicating, if it was a cache `HIT` or `MISS`.


=== "YAML"
  ```yaml
  cache:
    # Activate debug header.
    x_header: true
    # Set debug header name to 'X-Kache'.
    x_header_name: x-kache
    # Set default ttl for single items.
    default_ttl: 1200s

  ```

### Reference

| Directive         | Type      | Description |
|------------------ |---------- |------------ |
| `x_header`        | `bool`    | Activates the X-Cache debug header. If set to `true`, Kache will add a HTTP response header to each response indicating if it was served from cache or not (cache hit or miss). |
| `x_header_name`   | `string`  | Specifies the name of the X-Cache debug header. For example, if set to `x-kache` and in case of a cache hit, the response will contain an additional HTTP header with `X-Kache` as key and `HIT` as value. Default is 'X-Kache'. |
| `default_ttl`     | `string`  | Is the default TTL (time-to-live) for cached items. The value is a duration string. A duration string is a possibly signed sequence of decimal numbers, each with optional fraction and a unit suffix, such as "300ms", "1.5h", or "2h45m". Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h". TTL must be greater than 0. |



[^1]:
    [RFC7234 -- HTTP Caching](https://httpwg.org/specs/rfc7234)
    

[^2]:
    [RFC7234 -- Calculating freshness lifetime](https://httpwg.org/specs/rfc7234.html#calculating.freshness.lifetime)