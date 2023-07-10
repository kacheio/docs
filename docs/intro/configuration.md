# Configuration

Kache can be easily configured with a YAML-based configuration file. 

The configuration contains everything that defines how HTTP requests and responses are handled and cached. Elements of the configuration include defining and configuring listeners as endpoints to which downstream clients can connect and upstream target hosts to which Kache connects.

The HTTP caching element contains the configuration options for the HTTP caching filter or middleware that sits between listeners and upstream targets, and the provider element that configures the actual storage backend.

Other elements allow to configure operations such as logging and exposing the Admin API.

For a full overview, check the configuration reference or the sample configuration file on GitHub:

* [Reference](../reference/index.md) 
* [Sample config](https://raw.githubusercontent.com/kacheio/kache/main/kache.sample.yml)


## Loading the configuration

The configuration file is loaded when the service is started. The path to the configuration file must be specified via the `config.file` command line flag:

```sh
./kache -config.file=kache.yml
```

While the service is running, the configuration values can be changed temporarily or permanently. Currently, only changes affecting the HTTP cache can be applied at runtime and without completely restarting the service.

## Changing the configuration

### Temporary

Before applying permanent changes, it is advisable to test the new configuration before carefully rolling it out to production. Changing configuration parameters using the [Cache Management API](../reference/api_specification.md#cache-configuration) is an excellent way to test configuration changes before applying them permanently. Items and values that are changed in this way do not survive a restart of the service.

### Permanent

In order to make and apply permanent changes to the service's configuration, the corresponding file that was loaded when the service was started must be edited and reloaded. Reloading the configuration can be done in two ways: either by explicit reloading, sending a SIGHUP to the running process, or by enabling automatic reloading when Kache is started.

## Reloading the configuration

To apply configuration changes to the configuration file without completely restarting the running service, Kache listens for the SIGHUP signal. Sending the signal to the running process triggers the reloading of the configuration file. The new configuration is applied only if something has changed.

```sh
kill -SIGHUP <pid>
```

## Auto-reloading

It is possible to automatically reload the configuration file if it has changed. Automatic reloading can be enabled at service startup via the `--config.auto-reload` flag . If enabled, Kache will watch the specified configuration file for changes and automatically reloads if it has changed. The watch interval can be specified with the `--config.watch-interval=<duration>` flag, where `<duration>` must be a valid duration string, e.g. `5s`. If auto-reload is enabled, but no watch interval is specified, the default interval is `10s`.

```sh
./kache --config.file=kache.yml --config.auto-reload --config.watch-interval=1s
```
