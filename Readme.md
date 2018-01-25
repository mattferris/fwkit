fwkit
=====

A modular approach to iptables-based firewall rulesets.

*fwkit* is primarily designed around the concept of *roles* and *services*. Roles maintain a collection of enabled services. A ruleset is compiled based on the services enabled for the *active* role. Roles can be customized to suit any need, and a number of roles are configured by default.

You can view a list of available roles, and set one of them to be the active role.

```
# fwkit role list
  file-server
  fog
  nextcloud
  open
  samba-ad-dc
  web-server

# fwkit role set file-server
role changed: samba-ad-dc -> file-server

# fwkit role list
* file-server
  fog
  nextcloud
  open
  samba-ad-dc
  web-server
```

Now the active role has been set to `file-server`, and the role listing indicated this.

You can view the services enabled for the role.

```
# fwkit service list

LOCAL SERVICES
Active Name           Description

     * cups           CUPS (IPP); 631/tcp, 631/udp
     * dhcp           Dynamic Host Configuration Protocol; 67/udp, 68/udp
       dns            Domain Name Service; 53/tcp, 53/udp
       ftp            File Transfer Protocol; 20/tcp, 21/tcp, 1024-65535/tcp
       http           Hypertext Transfer Protocol; 80/tcp
       https          Hypertext Transfer Protocol over TLS; 443/tcp
       kerberos       Kerberos v5; 88/tcp. 88/udp
       kpasswd        Kerberos v5 kpasswd; 464/tcp, 464/udp
       ldap           Lightweight Directory Access Protocol; 389/tcp, 389/udp
       ldaps          Lightweight Directory Access Protocol over TLS; 636/tcp
       msgc           Microsoft Global Catalog (Active Directory); 3268/tcp
       msgc-ssl       Microsoft Global Catalog over TLS (Active Directory); 3269/tcp
       msrpc          Microsoft RPC Dynamic Ports; 1024-1300/tcp, 49152-65535/tcp
     * ntp            Network Time Protocol; 123/udp
       radacct        RADIUS Accounting; 1813/tcp, 1813/udp
     * radius         RADIUS Authentication; 1812/tcp, 1812/udp
     * rpcls          Microsoft End Point Mapper (DCE/RPC Locator Service); 135/tcp
     * smb            Server Message Block (SMB); 445/tcp
     * smtp           Simple Message Transfer Protocol; 25/tcp
     * ssh            Secure Shell Server; 22/tcp
     * tftp           Trivial File Transfer Protocol; 69/udp

REMOTE SERVICES
Active Name           Description

     * dns            Domain Name Service; 53/tcp, 53/udp
     * http           Hypertext Transfer Protocol; 80/tcp
       https          Hypertext Transfer Protocol over TLS; 443/tcp
     * kerberos       Kerberos v5; 88/tcp, 88/udp
     * kpasswd        Kerberos v5 kpasswd; 464/tcp, 464/udp
       ldap           Lightweight Directory Access Protocol; 389/tcp, 389/udp
       ldaps          Lightweight Directory Access Protocol over TLS; 636/tcp
       msgc           Microsoft Global Catalog (Active Directory); 3268/tcp
       msgc-ssl       Microsoft Global Catalog over TLS (Active Directory); 3269/tcp
       msrpc          Microsoft RPC Dynamic Ports; 1024-1300/tcp, 49152-65535/tcp
     * ntp            Network Time Protocol; 123/udp
       rpcls          Microsoft End Point Mapper (DCE/RPC Locator Service); 135/tcp
       smb            Server Message Block (SMB); 445/tcp
     * smtp           Simple Message Transfer Protocol; 25/tcp
     * ssh            Secure Shell Server; 22/tcp
```

A listing of all available local and remote services is display, with a description and port listing.

Enabling and disabling services is easy.

```
# fwkit service enable local ldap ldaps
enabled local service(s) ldap ldaps
```

Once you've configured the appropriate services, you can load the ruleset.

```
# fwkit rules load
compiled rules into /var/lib/fwkit/compiled.rules
loading compiled rules...done
```

Unloading the ruleset is simple, and *fwkit* caches the ruleset for later.

```
# fwkit rules unload
rules unloaded
```

Subsequent calls to `fwkit rules load` will load the cached ruleset. If you makes changes to the enabled services, you'll need to clean the cache before loading the ruleset to see your changes.

```
# fwkit rules unload && fwkit rules clean && fwkit rules load
```

There is a handy shortcut for all three actions.

```
# fwkit reload
```

Customizing Your Ruleset
------------------------

Beyond simply enabling or disabling services, there are a few options for customizing rulesets further. There are a number of files that are included during compilation. To get an understanding of how these files interact, here's an overview of *fwkit*'s config structure.

```
/etc/fwkit/
    role.active -> roles/file-server
    roles/
        file-server/
            description
            policy.rules
            zones/
                default/
                   default_action
                   description
                   services/
                       local/
                   services/
                       remote/
        ...
    rules.d/
        pre/
            0default.rules
        post/
    services/
        local/
            ...
        remote/
            ...
```

### policy.rules

This file is included before all others during the compilation process and is intented to configure chain policies. For example, you may want to set the `INPUT` and `OUTPUT` chain policies to `DROP`.

```
# policy.rules - set default policy to drop
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

### zones/<zone>/default_action

This file specifies the default action of traffic for the zone. Values are `allow` or `deny`.

### rules.d

This directory contains two sub-directories: `pre` and `post`. Files in the `pre` sub-directory are included before service rules, while files in the `post` sub-directory are included after the service rules. Files in either of these sub-directories must be suffixed with `.rules` in order to be included.

```
# rules.d/pre/0default.rules
# stateful traffic
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# loopback traffic
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# icmp traffic
iptables -A INPUT -p icmp -j ACCEPT
iptables -A OUTPUT -p icmp -j ACCEPT
```

### .service

Service definitions - those that appear when running `fwkit service list` - are defined in `services/local` and `services/remote` respectively. `services/local` contains rules for locally available services, while `services/remote` contains rules for remotely available services. Service definitions *should* contain a comment at the top of the file with information regarding the server. This information is used when listing the services.

```
# fwkit - local/dns.service - Domain Name Service; 53/tcp, 53/udp
iptables -A INPUT -p tcp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
```

When a service is enabled, it is symlinked into the active role's `zones/<zone>/services/local` or `zones/<zone>/services/remote` sub-directory.

```
# ls /etc/fwkit/role.active/zones/default/services/local
lrwxrwxrwx 1 root root 35 Jul 20 18:01 dns.service -> /etc/fwkit/role.active/zones/default/services/local/dns.service
lrwxrwxrwx 1 root root 35 Jul 20 18:01 ntp.service -> /etc/fwkit/role.active/zones/default/services/local/ntp.service
lrwxrwxrwx 1 root root 36 Jul 20 18:01 smtp.service -> /etc/fwkit/role.active/zones/default/services/local/smtp.service
lrwxrwxrwx 1 root root 35 Jul 20 18:37 ssh.service -> /etc/fwkit/role.active/zones/default/services/local/ssh.service
```

If service definition files have the execute bit set, then fwkit attempts to run the definition as a script. The script is passed two arguments: the zone the service is being enabled for, and the location of the service (*local* or *remote*). The script's output is captured and is included in the compiled rule data. fwkit includes an utility that called `fwksrv` which is used to allow for YAML service definitions. A service definition using a YAML format looks like:

```yaml
#!/usr/sbin/fwksrv
# fwkit - local/dns.service - Domain Name Service; 53/tcp, 53/udp

services:
  dns:
    description: Domain Name Service
    connections:
      -
        protocol: tcp
        listen_port: 53
      -
        protocol: udp
        listen_port: 53
```

Configuration
-------------

Configuration values are set/unset/viewed using the `fwkit config` command and come in two flavours: core configuration options, and ruleset variables. You can view the full list of configured values like so:

```
# fwkit config get
core.interpreter = /bin/sh
vars.opt_log_dropped = no
```

The above values are defined by default. `core.interpreter` controls the command used to parse the compiled ruleset. `vars.opt_log_dropped` defines the variable `opt_log_dropped` for use within the ruleset and is ultimately used to determine if logging rules should be added for dropped packets.

```
# fwkit - rules.d/post/99log-dropped.rules

if [ "$opt_log_dropped" = "yes" ]; then
    iptables -A INPUT -j LOG
    iptables -A OUTPUT -j LOG
fi

```

To set a configuration value, use `fwkit config set`.

```
# fwkit config set vars.foo bar
# fwkit config get
core.interpreter = /bin/sh
vars.opt_log_dropped = no
vars.foo = bar
```

To remove a configuration value, use `fwkit config unset`

```
# fwkit config unset vars.foo
# fwkit config get
core.interpreter = /bin/sh
vars.opt_log_dropped = no
```

### Runtime Variables

fwkit provides some runtime variables for use within your ruleset. These variables provide easy access to each network interface's address and network address. These variables take the format `if_<ifname>_addr` and `if_<ifname>_net` and can be used within a ruleset to provide some more dynamic rules.

```
iptables -A INPUT -p icmp -s $if_eth0_net -j ACCEPT
iptables -A OUTPUT -p icmp -d $if_eth0_net -j ACCEPT
```

Runtime variables can also be referenced in config values for further abstracton.

```
# fwkit config set vars.lan_net \$if_eth0_net
```

The above ICMP rules can now be written in an interface agnostic way, making them more portable across hosts.

```
iptables -A INPUT -p icmp -s $lan_net -j ACCEPT
iptables -A OUTPUT -p icmp -d $lan_net -j ACCEPT
```

Simple Port Forwarding
----------------------

The rules file `rules.d/pre/10dnat.rules` helps to make simple port forwarding easy to setup. Variables starting with `dnat_` are assumed to be port forwarding specifications, and the appropriate iptables command are generated to fulfill the specification. Port forward specifications have the format:

```
proto/src_addr:src_port/dst_addr:dst_port
```

`src_addr` and `dst_port` are optional. In cases where `src_addr` isn't defined, `src_port` must still be prefixed with a colon. Here are some examples.

- `tcp/:2022/10.10.5.17:22` - redirect incoming SSH connections on port 2202 to port 22 on host 10.10.5.17
- `udp/172.16.42.3:53/172.16.42.13` - redirect incoming DNS requests to local address 172.16.42.3 to host 172.16.42.13 (port 53 is implied as not destination port was specified)

To setup the port forward, run `fwkit config set`.

```
# fwkit config set vars.dnat_ssh tcp/:2202/10.10.5.17:22
# fwkit config set vars.dnat_dns udp/172.16.42.3:53/172.16.42.13
# fwkit config get
...
vars.dnat_ssh = tcp/:2202/10.10.5.17:22
vars.dnat_dns = udp/172.16.42.3:53/172.16.42.13
```

Rules automatically generated by this simple port forwarding setup are commented to make troubleshooting easier.
