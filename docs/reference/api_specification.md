# API

Kache provides an API for cache control and managment and exposes a number of informations through the API, such as the configuration of all listeners, targets, etc. The admin API can be enabled via the configuration.

## Security
Enabling the API will expose all configuration elements, including sensitive data.

Kache supports the [configuration of an access control list](#acl), which grants or denies access to API resources and objects. 

In production, it should be further protected by application level protection mechanisms, e.g., secured by authentication and authorization. Or, by transport level protection mechanisms such as NOT publicly exposing the API's port, but keeping it restricted to internal networks (principle of least privilege, applied to networks).


## Configuration

### Port

This enables the API and exposes its endpoints via the specified port.

=== "YAML"
  ``` yaml
  api:
    # API port.
    port: 1338
  ```

### Prefix

Configure a custom path prefix for all API endpoints, other than debug.

=== "YAML"
  ``` yaml
  api:
    # Enable API at port.
    port: 1338
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
    port: 1338
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
