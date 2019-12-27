# vpn_manager
Shell script for the deployment of vpn connections inside their own network namespaces.
## Concept
vpn_manager utilizes Linux's network namespace support to deploy a vpn connection that runs in an isolated network environment.
The advantage is that the vpn connection must not be used by all processes on the running machine. Instead the user
can decide which programs should use the vpn by launching them in the network namespace.
vpn_manager reads a user-supplied configuration file containing network settings and paths to an openvpn configuration.
Using that, vpn_manager can manage user-defined network namespaces and openvpn instances running in them.
After vpn_manager successfully started a network namespace, it provides a way to launch processes
using it through its _run_ command.

## Configuration file format
### Syntax
A configuration file consists of multiple variable initializations separated by new lines.
Every line must contain exactly one variable. Empty lines are ignored. Comments are not available.
Identifiers must be capitalized.

Variables must be defined like shell variables. That means, no spaces between the identifier and
the equal sign and the value. A correct variable initialization looks like this:

_VAR=value_

### Valid configuration file variables

**GW4**  
The IPv4 gateway the network namespace will use to connect to the internet.

**IP4**  
The IPv4 address and subnet the network namespace will use in the local network, CIDR notation must be used.

**NAMESPACE**  
Defines the name for the network namespace the VPN connection will use.

**NETDEV**  
The network bridge where the virtual ethernet device for the network namespace
will connect to. Usually this is _br0_.

**RESOLV**  
Path to a _resolv.conf_ file the network namespace will use for DNS resolution. Optional.  
Hint: You may leave this out and deploy your _resolv.conf_ at _/etc/netns/$NAMESPACE/resolv.conf_.

**VPN_AUTH**  
Path will be the argument for openvpn's _--auth-user-pass_ switch. Optional.

**VPN_CONFIG**  
Path to the _.ovpn_ configuration file you want to use with openvpn.

**VPN_RUNDIR**  
Path will be the argument for openvpn's _--cd_ switch. The VPN_RUNDIR must be specified before
any other VPN_* option in the config file. Optional.

## Commands
### start
Start the vpn connection specified in _configfile_.

**Syntax:**  
vpn_manager **start** _CONFIGFILE_

**Example:**  
vpn_manager **start** _/etc/vpn_config/myvpn.conf_
### stop
Stop the vpn connection specified in _configfile_. If there are processes left using the VPN namespace, it fails.

**Syntax:**  
vpn_manager **stop** _CONFIGFILE_

**Example:**  
vpn_manager **stop** _/etc/vpn_config/myvpn.conf_
### list
List all VPN connections started via vpn_manager

### run
Run a command through the specified vpn.

**Syntax:**  
vpn_manager **run** _NAMESPACE \[-u|--user USER\] \[-b|--background\] COMMAND_

**Options:**  
\-u, \-\-user _USER_: Run _COMMAND_ as user _USER_.  
\-b, \-\-background: Run _COMMAND_ in the background and detach it from the shell.

## Known issues

- If *systemd-resolved* is running, the name resolution of programs that are aware of *systemd-resolved* will fail inside of network namespaces.

© 2019 Jörg Walter
