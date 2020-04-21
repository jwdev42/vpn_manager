# vpn\_manager
Shell script for the deployment of vpn connections inside their own network namespaces.
## Concept
vpn\_manager utilizes Linux's network namespace support to deploy a vpn connection that runs in an isolated network environment. The advantage is that the vpn connection must not be used by all processes on the running machine. Instead the user can decide which programs should use the vpn by launching them in the network namespace. vpn\_manager reads a user-supplied configuration file containing network settings and paths to an openvpn configuration. Using that, vpn\_manager can manage user-defined network namespaces and openvpn instances running in them. After vpn\_manager successfully started a network namespace, it provides a way to launch processes using it through its _run_ command.

## Configuration files

Every openvpn connection that is managed by vpn_manager needs its own configuration file. These configuration files must be placed inside the directory _/etc/vpn\_manager_ and must have the file extension "_.conf_".

### Configuration file format
#### Syntax
A configuration file consists of multiple variable initializations separated by new lines. Every line must contain exactly one variable. Empty lines are ignored. Comments are not available. Identifiers must be capitalized.

Variables must be defined like shell variables. That means, no spaces between the identifier and the equal sign and the value. A correct variable initialization looks like this:

_VAR=value_

#### Valid configuration file variables

**GW4**  
The IPv4 gateway the network namespace will use to connect to the internet.

**IP4**  
The local IPv4 address and the local subnet to be used, CIDR notation is mandatory.  
Clarification: The local network namespace that is created for the vpn connection needs to have its own dedicated ip address. This is the address you specify here.

**NETDEV**  
The network bridge where the virtual ethernet device for the network namespace will connect to. Usually this is _br0_.

**RESOLV**  
Path to a _resolv.conf_ file the network namespace will use for DNS resolution. Optional.  
Hint: You may leave this out and deploy your _resolv.conf_ at _/etc/netns/$NAMESPACE/resolv.conf_.

**VPN_AUTH**  
Path will be the argument for openvpn's _--auth-user-pass_ switch. Optional.

**VPN_CONFIG**  
Path to the _.ovpn_ configuration file you want to use with openvpn.

**VPN_RUNDIR**  
Path will be the argument for openvpn's _--cd_ switch. The VPN_RUNDIR must be specified before any other VPN_* option in the config file. Optional.

#### Namespace selection

The namespace is determined by the name of the configuration file without its file extension. If the configuration file is stored at _/etc/vpn\_manager/myvpn.conf_, the namespace will be _myvpn_.

## Commands
### start
Start the vpn connection by giving its namespace as an argument.

**Syntax:**  
vpn\_manager **start** _namespace_

**Example:**  
vpn\_manager **start** _myvpn_
### stop
Stop the vpn connection that runs in the namespace _namespace_, remove the associated network namespace afterwards. If there are processes left using the VPN namespace, it fails.

**Syntax:**  
vpn\_manager **stop** _namespace_

**Example:**  
vpn\_manager **stop** _myvpn_
### list
List all VPN connections started via vpn\_manager

### run
Run a command through the specified vpn.

**Syntax:**  
vpn\_manager **run** _NAMESPACE \[-u|--user USER\] \[-b|--background\] COMMAND_

**Options:**  
\-u, \-\-user _USER_: Run _COMMAND_ as user _USER_.  
\-b, \-\-background: Run _COMMAND_ in the background and detach it from the shell.

## Troubleshooting

### Name resolution does not work

- Only programs that are _aware_ of network namespaces use the designated _resolv.conf_ file. Programs that are not aware of network namespaces still use _/etc/resolv.conf_. If your first nameserver in _/etc/resolv.conf_ is not an ipv4 server, the name resolution will fail.
- The _hosts_ directive in your _nsswitch.conf_ file might not be configured to use dns, but some other plugin that is not compatible with vpn\_manager.

© 2020 Jörg Walter
