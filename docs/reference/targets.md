# Targets

Targets are a group of logically similar upstream hosts that Kache connects to. An upstream host receives connections and requests from Kache and returns responses.

### Configuration

The following example defines two upstream hosts identified by a unique name. The upstream target's address (`addr`) is a network location, Kache can connect and send requests to.

=== "YAML"
  ``` yaml
  upstreams:
    # Upstream service 1
    - name: service1
      addr: "<ip-service-1>:<port-service-1>"
      path: "path/to/service/1"

    # Upstream service 2
    - name: service2
      addr: "<ip-service-2>:<port-service-2>"
  ```


### Reference

| Directive   | Type        | Description                          |
| ----------- | ----------- | ------------------------------------ |
| `name`      | `string`    | The unique identifier of the upstream target.  |
| `addr`      | `string`    | The network location Kache can connect and send requests to. Typically a valid URL consisting of the private IP and Port of an upstream service.  |
| `path`      | `string`    | The path prefix used to match the upstream target. Path is forwarded as-is, hence the service is expected to listen on the specified path. |

