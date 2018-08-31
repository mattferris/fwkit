Changelog
=========

0.5
---

* Refactored into plumbing commands (`fwk-*`) and a procelain command (`fwkit`)
* Service definitions are now copied when enabled instead of symlinked to allow for customization
* ACLs are now generic, can be applied to multiple services, and support default actions
* ACL are now modified directly, instead of via the CLI commands
* Fixed iptables commenting
* Fixed bugs/typos in service definitions
* Added service definition for NetBIOS

0.4.1
-----

* Fixed systemd unit file, startup was caching empty ruleset and then loading empty (cached) ruleset

0.4
---

* Added support for zones
* Dynamic service definitions
* Service ACLs
* Improved command-line help output
* Removed pre-packaged roles in favour of post-install customization
* Role management now available via command-line
* IPv6 support via ip6tables
* Support for granular rule unloading (avoids simply flushing the built-in chains)

0.3
---

* Update systemd service definition (github issue #7, #9)
* Added remote RDP service and enabled by default on the file-server role (github issue #8)
* Command-line tweaks
* Updated readme
* The fwkit command can now be easily referenced from within rulesets
* Introduced simple port forwarding system
* Added man page
