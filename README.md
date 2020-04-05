oVirt Grafana for DWH
=================

This role configure Grafana for oVirt DWH DB.

- Get DB variables
- Create read-only user for ovirt_engine_history db and Grafana
- Install Grafana
- Configure Grafana

Target systems
--------------

* DWH on engine or remote

Requirements
------------

Preinstalled engine and DWH.


Running the role:
-----------------

From the role repo run the oVirt Grafana playbook:
```
# ansible-playbook configure-ovirt-grafana.yml --ask-vault-pass -vvv
```

Log into the Grafana portal and review the Dashboards.


Role Variables
--------------

```yml
---

```

Dependencies
------------

None

Example Playbook
----------------

```yml
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
