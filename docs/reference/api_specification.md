# API

Kache provides an API for cache control and managment and exposes a number of informations through the API, such as the configuration of all listeners, targets, etc. The admin API can be enabled via the configuration.

## Security
Enabling the API will expose all configuration elements, including sensitive data.

In production, it should be protected by application level protection mechanisms, e.g., secured by authentication and authorization. Or, by transport level protection mechanisms such as NOT publicly exposing the API's port, but keeping it restricted to internal networks (principle of least privilege, applied to networks).

## Configuration

### Port

This enables the API and exposes its endpoints via the specified port.

=== "YAML"
  ``` yaml
  api:
    # API port.
    port: 1338
  ```

### Path

Configure a custom path prefix for all API endpoints, other than debug.

=== "YAML"
  ``` yaml
  api:
    # Enable API at port.
    port: 1338
    # Customize path prefix, default is '/api'
    path: "/custom-api-prefix/"
  ```


### Debug

Activate endpoints for general debug informations.

=== "YAML"
  ``` yaml
  api:
    # API port.
    port: 1338
    # Activate debug endpoints.
    debug: true
  ```

## Endpoints

| Path                                  | Method | Description                          |
| ------------------------------------- | ------ | ------------------------------------ |
| `/api/cache/keys`                     | `GET`  | Returns the keys currently in the cache.  |
| `/api/cache/keys/<key>`               | `GET`  | Returns the key info about `<key>`.  |
| `/api/cache/keys/purge?key=<key>`     | `GET`  | Purges a `<key>` from the cache |
| `/api/version`                        | `GET`  | Returns the Kache version info.  |
| `/debug/vars`                         | `GET`  | See the [expvar](https://pkg.go.dev/expvar) Go documentation. |
| `/debug/pprof`                        | `GET`  | See the [pprof Index](https://golang.org/pkg/net/http/pprof/#Index) Go documentation. |
| `/debug/pprof/cmdline`                | `GET`  | See the [pprof Cmdline](https://golang.org/pkg/net/http/pprof/#Cmdline) Go documentation. |
| `/debug/pprof/profile`                | `GET`  | See the [pprof Profile](https://golang.org/pkg/net/http/pprof/#Profile) Go documentation. |
| `/debug/pprof/symbol`                 | `GET`  | See the [pprof Symbol](https://golang.org/pkg/net/http/pprof/#Symbol) Go documentation. |
| `/debug/pprof/trace`                  | `GET`  | See the [pprof Trace](https://golang.org/pkg/net/http/pprof/#Trace) Go documentation. |
