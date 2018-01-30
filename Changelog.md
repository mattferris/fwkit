Changelog
=========

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
