oVirt Grafana for DWH
=================

This role configure Grafana for oVirt DWH DB.

- get DB variables (database, user, password, ...)
- install Grafana
- Configure Grafana

Target systems
--------------

* dwh on engine or remote

Requirements
------------

Preinstalled engine and DWH.

Role Variables
--------------

```yaml
---

```

Dependencies
------------

None

Example Playbook
----------------

```yaml
---
- hosts: engine
  vars:

  roles:
    - ansible-role-ovirt-grafana
```

Author Information
------------------

Shirly Radco
sradco@redhat.com
