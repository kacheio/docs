# API

Kache provides an API for cache control and managment and exposes a number of informations through the API, such as the configuration of all listeners, targets, etc. The admin API can be enabled via the configuration.

## Security
Enabling the API will expose all configuration elements, including sensitive data.

Kache supports the [configuration of an access control list](#acl), which grants or denies access to API resources and objects. 

In production, it should be further protected by application level protection mechanisms, e.g., secured by authentication and authorization. Or, by transport level protection mechanisms such as NOT publicly exposing the API's port, but keeping it restricted to internal networks (principle of least privilege, applied to networks).

## Cache Management

### Purging a cache key

Purging a key from the cache is done by isssuing a `PURGE` request to the API root available at the 
specified API port. The cache key is retrieved from a custom request header `X-Purge-Key`. When specified
in wildcard syntax, all matching keys will be purged from the cache. If the specified key is an empty
string, all keys will be purged from the cache, similar to a cache flush.

``` sh
curl -v -X PURGE -H 'X-Purge-Key: <cache-key-pattern>' kacheserver:$PORT
```

### Flush cache keys

Flushing the cache deletes all the keys from the cache:

``` sh
curl -v -X DELETE 'kacheserver:$PORT/<api-prefix>/cache/flush'
```

### List cache keys

To list all the keys in the cache:

``` sh
curl -v -X GET 'kacheserver:$PORT/<api-prefix>/cache/keys'
```

### Cache configuration

The API provides endpoints to render and update the current cache configuration. Updating the cache configuration via the API currently only updates the runtime configuration and does not keep the changes in the specified configuration file that is loaded when kachserver is started. Thus, the changes are not preserved when the service is restarted.

Rendering the configuration:
```sh
curl -v -X GET 'kacheserver:$PORT/<api-prefix>/cache/config'
```

Updating the configuration:

??? example "Example: Update cache configuration"
    === "cURL"
    ```sh
    curl --location --request PUT 'kacheserver:$PORT/<api-prefix>/cache/config/update' \
        --header 'Content-Type: application/json' \
        --data '{
            "x_header": true,
            "x_header_name": "x-kache",
            "default_ttl": "1200s",
            "default_cache_control": "max-age=120s",
            "force_cache_control": false,
            "timeouts": [
                {
                    "path": "/archive",
                    "ttl": 86400s
                },
                {
                    "path": "^/assets/([a-z0-9].*).css",
                    "ttl": 30s
                }
            ],
            "exclude": {
                "path": [
                    "^/admin",
                    "^/test",
                    "^/.well-known/acme-challenge/(.*)"
                ],
                "header": {
                    "x_requested_with": "XMLHttpRequest"
                },
                "content": [
                    {
                        "type": "application/javascript|text/css|image/.*",
                        "size": 10000
                    }
                ]
            }
        }'
    ```



## Configuration

### Port

This enables the API and exposes its endpoints via the specified port.

=== "YAML"
  ``` yaml
  api:
    # API port.
    port: 6067
  ```

### Prefix

Configure a custom path prefix for all API endpoints, other than debug, version, and purge.

=== "YAML"
  ``` yaml
  api:
    # Enable API at port.
    port: 6067
    # Customize path prefix, default is '/api'.
    prefix: "/custom-api-prefix/"
  ```

### ACL

Configure an access control list to grant access to resources only for the specified IP addresses. If 
`acl` is specified, access is granted only for requests originating from IPs on the list. All others 
will be denied. If the list is empty or the item is not specified in the configuration at all, 
any request is allowed.

=== "YAML"
  ``` yaml
  api:
    # Enable API at port.
    port: 6067
    # Access control list. If empty or not specified at all, 
    # any request is allowed to access API resources and objects.
    acl: "127.0.0.1, 10.22.0.0"
  ```

### Debug

Activate endpoints for general debug informations.

=== "YAML"
  ``` yaml
  api:
    # API port.
    port: 6067
    # Activate debug endpoints.
    debug: true
  ```

## Endpoints

| Path                                  | Method    | Description                          |
| ------------------------------------- | --------- | ------------------------------------ |
| `/`                                   | `PURGE`   | Purges a `<key>` from the cache. The cache key is retrieved from a custom request header `X-Purge-Key` and support wildcard matching.     |
| `/<api-prefix>/cache/keys`            | `GET`     | Returns the keys currently in the cache.  |
| `/<api-prefix>/cache/keys/<key>`      | `GET`     | Returns the key info about `<key>`.  |
| `/<api-prefix>/cache/flush`           | `DELETE`  | Flushes all keys from the cache.  |
| `/<api-prefix>/cache/config`          | `GET`     | Renders the current cache config.  |
| `/<api-prefix>/cache/config/update`   | `PUT`     | Updates the current cache config.  |
| `/version`                            | `GET`     | Returns the Kache version info.  |
| `/debug/vars`                         | `GET`     | See the [expvar](https://pkg.go.dev/expvar) Go documentation. |
| `/debug/pprof`                        | `GET`     | See the [pprof Index](https://golang.org/pkg/net/http/pprof/#Index) Go documentation. |
| `/debug/pprof/cmdline`                | `GET`     | See the [pprof Cmdline](https://golang.org/pkg/net/http/pprof/#Cmdline) Go documentation. |
| `/debug/pprof/profile`                | `GET`     | See the [pprof Profile](https://golang.org/pkg/net/http/pprof/#Profile) Go documentation. |
| `/debug/pprof/symbol`                 | `GET`     | See the [pprof Symbol](https://golang.org/pkg/net/http/pprof/#Symbol) Go documentation. |
| `/debug/pprof/trace`                  | `GET`     | See the [pprof Trace](https://golang.org/pkg/net/http/pprof/#Trace) Go documentation. |
