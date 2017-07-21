fwkit
=====

A modular approach to iptables-based firewall rulesets.

*fwkit* is primarily designed around the concept of *roles* and *services*. Roles maintain a collection of enabled services. A ruleset is compiled based on the services enabled within the current role. Roles can be customized to suit any need, and a number of roles are configured by default.

Roles
-----

Get a list of available roles.

```
# fwkit role list
  file-server
  fog
  nextcloud
  open
* samba-ad-dc
  web-server
```

The asterisk denotes the active role. You can also specifically request the active role.

```
# fwkit role get
samba-ad-dc
```

Change the active role.

```
# fwkit role set file-server
role changed: samba-ad-dc -> file-server
```

Services
--------

Services are broken into two groups: local and remote. Local services are those host locally, while remote services are services that are not hosted locally.

List the services for the active role.

```
# fwkit service list

Local services
    cups
    dhcp
  * dns
    ftp
    http
    https
  * kerberos
  * kpasswd
  * ldap
  * ldaps
  * msgc
  * msgc-ssl
  * msrpc
  * ntp
    radacct
    radius
  * rpcls
    smb
  * smtp
  * ssh
    tftp

Remote services
  * dns
  * http
    https
  * kerberos
  * kpasswd
  * ldap
  * ldaps
  * msgc
  * msgc-ssl
  * msrpc
  * ntp
  * rpcls
  * smb
  * smtp
  * ssh
```

Disable the local ssh service.

```
# fwkit service disable local ssh
disabled local service(s) ssh
```

Enable the local ssh service as well as the local http service.

```
# fwkit service enable local ssh http
enabled local service(s) ssh http
```

Rules
-----

Load the ruleset.

```
# fwkit ruleset load
compiled rules into /var/lib/fwkit/compiled.rules
loading compiled rules...done
```

Unload the ruleset.

```
# fwkit rules unload
rules unloaded
```

Unloading the rules caches the ruleset (and all counter data). Loading the ruleset again will cause the cached ruleset to be loaded instead of compiling the ruleset from scratch. This is indicated when loading the rulset.

```
# fwkit rules load
loading cached rules...done
```

If services are enabled or disable, or the role changes, you'll want to clear the cached rules (`clean`) and reload the rulset.

```
# fwkit rules unload && fwkit rules clean && fwkit rules load
```

Or do a `reload`, which does the above, just in one simple command.

```
# fwkit rules reload
```
