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

1. On the Manager machine, copy /etc/config.yml.example to /etc/config.yml.d/config.yml:
```
# cp /etc/ovirt-grafana/config.yml.example /etc/ovirt-grafana/config.yml.d/config.yml
```
2. Update the values of /etc/ovirt-grafana/config.yml.d/config.yml to match the details of your specific environment:
```
# vi /etc/ovirt-grafana/config.yml.d/config.yml
```

3. On the Manager machine, copy /etc/secure_vars.yaml.example to /etc/config.yml.d/secure_vars.yaml:
```
# cp /etc/secure_vars.yaml.example /etc/config.yml.d/secure_vars.yaml
```

4. Update the values of /etc/ovirt-engine-metrics/secure_vars.yaml to match the details of your specific environment:
```
# vi /etc/config.yml.d/secure_vars.yaml
```

5. Encrypt secure_vars.yaml file
```
# ansible-vault encrypt /etc/config.yml.d/secure_vars.yaml
```

6. Go to ovirt-engine-metrics repo:
```
# cd /usr/share/ovirt-grafana/playbooks
```

7. Run the oVirt Grafana installation playbook that installs and configures Grafana for oVirt DWH:
```
# ansible-playbook configure-ovirt-grafana.yml --ask-vault-pass -vvv
```

8. Log into the Grafana portal and review the Dashboards.


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
