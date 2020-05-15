Role Name
=========

This role manages EdgeOS firewall settings.

Requirements
------------

Hosts that use this role will need to use a connection type of network_cli, have "enable" as the become method, and ansible_network_os set to "edgeos"


Role Variables
--------------

firewall_process_rules: When set to True, firewall rules will be processed. (Default: True)
firewall_process_zones: When set to True, firewall zones will be processed. (Default: True)
firewall_cleanup: When set to True, any firewall settings that do not exist in the firewall_rules variable will be deleted. (Default: False)
firewall_combine_apply: When set to True, all firewall settings will be set at the same time.  This includes cleanup, firewall rules, and firewall zones. (Default: False)
firewall_commit: When set to True, changes will be committed (but not saved.)  A value of False will allow you to see what changes will be made without committing them. (Default: False)
firewall_rules: A list containing firewall rules.  (Default: empty list)

If a rule set uses ipv6, the rule set name should end in "-v6"

Example:
```
firewall_rules:
  - name: lan-local
    properties:
      - default-action-drop
    rules:
      - name: 200
        properties:
          - action accept
          - destination port 80,443
          - log enable
          - protocol tcp
      - name: 800
        properties:
          - action accept
          - destination port 22
          - log enable
          - protocol tcp
  - name: lan-local-v6
    rules:
      - name: 101
        properties:
          - action accept
          - icmpv6 type echo-request
          - protocol ipv6-icmp
      - name: 102
        properties:
          - action accept
          - icmpv6 type neighbor-solicitation
          - protocol ipv6-icmp
      - name: 103
        properties:
          - action accept
          - icmpv6 type neighbor-advertisement
          - protocol ipv6-icmp
      - name: 104
        properties:
          - action accept
          - icmpv6 type router-advertisement
          - protocol ipv6-icmp
      - name: 105
        properties:
          - action accept
          - icmpv6 type router-solicitation
          - protocol ipv6-icmp
```

In order to make rules more reusable, you can make use of yaml anchors and aliases:
```
firewall_rule_aliases:
  - &rule_allow_icmp
    name: 100
    properties:
      - action accept
      - log enable
      - protocol icmp
  - &rule_allow_web
    name: 200
    properties:
      - action accept
      - destination port 80,443
      - log enable
      - protocol tcp
  - &rule_allow_ssh
    name: 800
    properties:
      - action accept
      - destination port 22
      - log enable
      - protocol tcp

firewall_rules:
  - name: lan-local
    rules:
      - *rule_allow_icmp
      - *rule_allow_web
      - *rule_allow_ssh
```

firewall_zones: A list containing firewall zones.  (Default: empty list)

Example:
```
firewall_zones:
  - name: wan
    properties:
      - default-action drop
      - interface eth0
  - name: lan
    properties:
      - default-action drop
      - interface eth1
  - name: local
    properties:
      - default-action drop
      - local-zone
```

global_firewall_settings: A list of settings that get applied to all firewall rule sets and firewall rules.  (Default: empty list)

Example:

```
global_firewall_settings:
  - properties:
      - default-action drop
      - enable-default-log
  - properties_v6:
      - default-action drop
      - enable-default-log
  - rules:
      - name: 1
        properties:
          - action accept
          - state established enable
          - state related enable
      - name: 2
        properties:
          - action drop
          - log enable
          - state invalid enable
  - rules_v6:
      - name: 1
        properties:
          - action accept
          - state established enable
          - state related enable
      - name: 2
        properties:
          - action drop
          - log enable
          - state invalid enable
```
Example Playbook
----------------
```
    - hosts: er4
      connection: network_cli
      gather_facts: False
      roles:
        - ansible-role-edgeos-firewall
```

License
-------

MIT
