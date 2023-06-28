# API

Kache offers a REST API into Kache, exposing endpoints for cache management and debug information.

For example, the following configuration exposes the API on the specified port and activates endpoints exposing debug informations.

=== "YAML"
  ``` yaml
  api:
    # API port.
    port: 1338
    # Activate debug endpoints.
    debug: true
  ```

The Kache API endpoints are generally available under path prefix `/api`, unless not specified a different prefix. A complete overview of the API and its endpoints is available [here](./api_specification.md).

### Reference

| Directive     | Type        | Description                          |
| -----------   | ----------- | ------------------------------------ |
| `port`        | `string`    | The port the API is available at.      |
| `prefix`      | `string`    | Adds a custom path prefix.           |
| `debug`       | `bool`      | Activates debug endpoints.           |