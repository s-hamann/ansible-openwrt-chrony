OpenWRT Chrony
==============

This role configures [Chrony](https://chrony-project.org/) on [OpenWRT](https://www.openwrt.org/) targets.
Chrony can be used as a NTP client or server, with or without Network Time Security (NTS) support.

Requirements
------------

This role has no special requirements on the controller.

It does, however, require a working [Python](https://www.python.org/) installation on the target system or [gekmihesg's Ansible library for OpenWRT](https://github.com/gekmihesg/ansible-openwrt) on the Ansible controller.

Role Variables
--------------

Chrony can run in client mode and in server mode.
In client mode, it keeps the system's time in sync with upstream NTP server.
Options that are relevant only for this mode start with `chrony_client_`.
In server mode, it acts as an upstream NTP server for other systems.
Options that are relevant only for this mode start with `chrony_server_`.

* `chrony_client_nts`  
  Whether to enable [NTS](https://tools.ietf.org/html/rfc8915) in the Chrony client, i.e. when the system gets the current time from NTP servers.
  When NTS is enabled, it is enabled for all servers and pools.
  Therefore, only server and pools with NTS support should be configured.
  Note that the `chrony-nts` package requires more space than the plain `chrony` package.
  On systems that are highly constrained for storage space, it may be better to disable NTS support.
  See also `chrony_server_nts`.
  Default is `true`.
* `chrony_client_servers`, `chrony_client_pools`, `chrony_client_peers`  
  A list of servers, pools and peers, respectively, to synchronize time with.
  Refer to the [Chrony documentation](https://chrony-project.org/doc/4.2/chrony.conf.html) for details on what the difference is.
  When `chrony_client_nts` is enabled, the default for `chrony_client_servers` is `time.cloudflare.com` and `chrony_client_pools` and `chrony_client_peers` are empty.
  When `chrony_server_nts` is disabled, the default for `chrony_client_pools` is `openwrt.pool.ntp.org` and `chrony_client_servers` and `chrony_client_peers` are empty.
* `chrony_client_minpoll`  
  The minimal interval between requests to servers/pools/peers as a power of 2 in seconds.
  For example, `5` would mean that the polling interval should not drop below 32 seconds.
  If not set, Chrony's internal default value is used.
* `chrony_client_maxpoll`  
  The maximum interval between requests sent to servers/pools/peers as a power of 2 in seconds.
  For example, `9` indicates that the polling interval should stay at or below 512 seconds.
  Default is `12` (4096 seconds - 1 hour, 8 minutes and 16 seconds).
* `chrony_client_makestep_threshold`  
  The threshold in seconds for the required time correction at which Chrony will step the system clock instead of gradually slewing it.
  Default is `1.0` seconds.
* `chrony_client_makestep_limit`  
  The limit for the number of stepped clock updates.
  Subsequent updates are slewed, even if the correction is larger than `chrony_client_makestep_threshold`.
  Default is `3`.
* `chrony_client_dhcp_ntp_server`  
  Whether Chrony should use the NTP server provided by the upstream DHCP server (if any).
  Default is `false` if `chrony_client_nts` is enabled and `true` otherwise.
* `chrony_server_interfaces`  
  A list of OpenWrt interfaces on which Chrony should act as a server, i.e. provide NTP service to other systems.
  Default is empty, i.e. the NTP server is disabled.
* `chrony_server_nts`  
  Whether to enable NTS in the Chrony server, i.e. when the system acts as an NTP server for other systems.
  See also `chrony_client_nts`.
  Default is `true` if `chrony_server_interfaces` is not empty and `false` if the server is disabled anyway.
* `chrony_server_cert`  
  Path to a PEM-encoded X.509 certificate for Chrony to use, including any intermediate certificates.
  The file needs to exist and be readable by the `chrony` user.
  Mandatory if `chrony_server_nts` is `true`, ignored otherwise.
* `chrony_server_key`  
  Path to the PEM-encoded private key file for the certificate.
  The file needs to exist and be readable by the `chrony` user.
  Mandatory if `chrony_server_nts` is `true`, ignored otherwise.
* `chrony_extra_groups`  
  A list of groups that the `chrony` system user is added to.
  This allows granting access to additional resources, such as the private key file when running a NTS server.
  All groups need to exist on the target system; this role does not create them.
  Empty by default.
* `chrony_extra_options`  
  A list of raw configuration lines to be added to `chrony.conf`.
  This allows full customization of the Chrony configuration that goes beyond the abstraction provided by this role.
  Refer to the [Chrony documentation](https://chrony-project.org/doc/4.2/chrony.conf.html) for valid directives.
  Optional.

Dependencies
------------

This role does not depend on any specific roles.

However, when running Chrony as a server with NTS support, an X.509 certificate and key are required.
This role does not handle deployment of these files.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
chrony_client_servers:
  - ntppool1.time.nl
  - nts.netnod.se
  - ptbtime1.ptb.de
  - virginia.time.system76.com
chrony_server_interfaces:
  - lan
chrony_server_cert: "/etc/{{ ansible_facts['hostname'] }}_fullchain.pem"
chrony_server_key: "/etc/{{ ansible_facts['hostname'] }}.key"
chrony_extra_groups:
  - tls
chrony_extra_options:
  - 'maxsamples 12'
  - 'refclock SOCK /var/run/chrony.ttyS0.sock'
  - 'clientloglimit 16384'
  - 'ratelimit interval 1 burst 16'
  - 'ntsratelimit interval 3 burst 1'
```

License
-------

MIT
