# Custom Victron Energy nodes for Node-Red

This module aims on providing custom node-red nodes for some of the most commonly used Victron Energy products. This makes it easier for end users to create complex automations without actually having to touch the devices' internals.

See [the initial node specification](documentation/specification.md) for custom victron-supported nodes.
The tracking issue can be found on [venus repository](https://github.com/victronenergy/venus/issues/378).

## Architecture

### Plugin Behavior

All the individual nodes (inputs / outputs) will use a singleton instance of a Victron Config Node to access the system dbus in a Venus device. The nodes will provide a easy-to-use interface for accessing various measurements and writing data to the system.

The following graph demonstrates the architecture of this plugin.

1. Upon initialization, the Victron Config Node initializes a VictronClient and SystemConfiguration instances. VictronClient connects to the Venus D-Bus and starts maintaining a cache of available services.

2. When a user modifies a node (e.g. battery node), the node fetches the available services (measurements) from the SystemConfiguration cache and renders relevant inputs to the edit view.

3. When a node is deployed, they either subscribe a message handler to the VictronClient or start publishing data to a desired D-Bus path.


![Architecture](documentation/images/architecture.png)

### Directory Structure

```
.
├── README.md
├── documentation
│   ├── images
│   │   ├── architecture.png
│   │   ├── example-tank-edit.png
│   │   └── example-use-case.png
│   └── specification.md
└── src
    ├── nodes
    │   ├── config-client.html
    │   ├── config-client.js        <-- global configuration node
    │   ├── input-battery.html
    │   ├── input-battery.js        <-- battery node
    │   ├── output-relay.html
    │   └── output-relay.js         <-- relay node
    └── services
        ├── dbus-listener.js        <-- github.com/sbender9/signalk-venus-plugin
        ├── systemconfiguration.js  <-- SystemConfiguration cache
        └── victronclient.js        <-- Victron Energy D-Bus Client
```

## Installation and Usage

In order to use the plugin, it needs to be locally installed:

1. install node-red on your system
2. cd to the node-red user directory, typically `~/.node-red`
3. install node-red-contrib-victron locally, `npm install /path/to/this/repository`
4. enable d-bus over tcp in your Venus device. Edit `/etc/dbus-1/system.conf` and add the following directly above `<policy context="default">`:

```
  <listen>tcp:host=0.0.0.0,port=78</listen>
  <auth>ANONYMOUS</auth>
  <allow_anonymous/>
```
5. environment variable `NODE_RED_DBUS_ADDRESS` needs to be set before running node-red in order to connect to the dbus instance. If it is not set, it defaults to localhost. For example `NODE_RED_DBUS_ADDRESS=192.168.1.1 node-red`. 

Further information on [nodered.org](https://nodered.org/docs/creating-nodes/first-node) and [github](https://github.com/sbender9/signalk-venus-plugin#plugin-installation--configuration).
6. you can optionally run the plugin with a DEBUG environment variable set, to see additional debug information printed on the shell.
