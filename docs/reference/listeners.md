# Listeners

A listener is a named network location (e.g., port, etc.) that can be connected to by downstream clients. Kache exposes one or more listeners that downstream hosts can connect to.

A listener is configured with a unique name and a network adress. The address defines a port, and optionally a hostname, on which kache listens for incoming connections.

### Configuration

For example, the following configuration exposes two named listeners, `web1` and `web2` listening on ports `:80` and `:1337`, respectively.

=== "YAML"
  ```yaml
  listeners:
    # Listener 1 named web1
    web1:
      addr: :80

    # Listener 2 named web2
    web2:
      addr: :1337
  ```

After the listeners are up and running, kache accepts connections and forwards them to defined upstream targets.

### Reference

| Directive     | Type        | Description                          |
| -----------   | ----------- | ------------------------------------ |
| `<name>`      | `token`     | Name variable is the unique name of the listener.  |
| `addr`        | `string`    | The network location exposed by Kache that can be connected to by downstream clients. It accepts a port, and optionally a hostname, in the format of `[host]:port`. |