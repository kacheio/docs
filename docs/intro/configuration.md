# Configuration

Kache can be easily configured with a YAML-based configuration file. 

The configuration contains everything that defines how HTTP requests and responses are handled and cached. Elements of the configuration include defining and configuring listeners as endpoints to which downstream clients can connect and upstream target hosts to which Kache connects.

The HTTP caching element contains the configuration options for the HTTP caching filter or middleware that sits between listeners and upstream targets, and the provider element that configures the actual storage backend.

Other elements allow to configure operations such as logging and exposing the Admin API.

For a full overview, check the configuration reference or the sample configuration file on GitHub:

* [Reference](../reference/index.md) 
* [Sample config](https://raw.githubusercontent.com/kacheio/kache/main/kache.sample.yml)


