timesync
========

This role installs and configures an NTP and/or PTP implementation to operate
as an NTP client and/or PTP slave in order to synchronize the system clock with
NTP servers and/or grandmasters in PTP domains. Supported NTP/PTP
implementations are chrony, ntp (the reference implementation) and linuxptp.

Role Variables
--------------

The variables that can be passed to this role are as follows:

```
# Name of the NTP implementation that will be installed and configured
# when one or more NTP servers are specified (default chrony)
ntp_implementation: chrony

# Name of the PTP implementation that will be installed and configured
# when one or more PTP domains are specified (default linuxptp)
ptp_implementation: linuxptp

# List of NTP servers
ntp_servers:
  - name: foo.example.com       # Hostname or address of the server
    minpoll: 4                  # Minimum polling interval (default 6)
    maxpoll: 8                  # Maximum polling interval (default 10)
    iburst: yes                 # Flag enabling fast initial synchronization
                                # (default no)
    pool: no                    # Flag indicating that each resolved address
                                # of the hostname is a separate NTP server
                                # (default no)

# List of PTP domains
ptp_domains:
  - number: 0                   # PTP domain number
    interfaces: [ eth0 ]        # List of interfaces in the domain
    delay: 0.000010             # Assumed maximum network delay to the
                                # grandmaster in seconds
                                # (default 100 microsecond)
    transport: UDPv4            # Network transport: UDPv4, UDPv6, L2
                                # (default UDPv4)
    udp_ttl: 1                  # TTL for UDPv4 and UDPv6 transports
                                # (default 1)

# Extra options that will be added to chrony.conf (optional)
chrony_extra_conf: |
  logdir /var/log/chrony
  log statistics

# Extra options that will be added to ntp.conf (optional)
ntp_extra_conf: |
  statistics loopstats peerstats

# Extra options that will be added to ptp4l.conf (optional)
ptp4l_extra_conf: |
  logging_level 7
```

Example Playbook
----------------

Install and configure ntp to synchronize the system clock with three NTP servers:

```
- hosts: targets
  vars:
    ntp_servers:
      - name: foo.example.com
        iburst: yes
      - name: bar.example.com
        iburst: yes
      - name: baz.example.com
        iburst: yes
    ntp_implementation: ntp
  roles:
    - timesync
```

Install and configure linuxptp to synchronize the system clock with a
grandmaster in PTP domain number 0, which is accessible on interface eth0:

```
- hosts: targets
  vars:
    ptp_domains:
      - number: 0
        interfaces: [ eth0 ]
  roles:
    - timesync
```

Install and configure chrony and linuxptp to synchronize the system clock with
multiple NTP servers and PTP domains for a highly accurate and resilient
synchronization:

```
- hosts: targets
  vars:
    ntp_servers:
      - name: foo.example.com
        maxpoll: 6
      - name: bar.example.com
        maxpoll: 6
      - name: baz.example.com
        maxpoll: 6
    ptp_domains:
      - number: 0
        interfaces: [ eth0, eth1 ]
        transport: L2
        delay: 0.000010
      - number: 1
        interfaces: [ eth2 ]
        transport: UDPv4
        delay: 0.000010
    chrony_extra_conf: |
      logdir /var/log/chrony
      log tracking
  roles:
    - timesync
```
