rhbk_poc
========

This role will create a 2 nodes RHBK cluster with a single postgreSQL DB, and a HAPROXY in front of the RHBK nodes.

Requirements
------------

This role assumes your RHEL servers are joined to an IDM(FreeIpa) domain.

It use the following ansible inventory.

| Group name | Description |
|:-----------|:------------|
|`sso_haproxy`| host to install HAproxy load-balancer |
|`sso_servers`| hosts to install RHBK |
|`sso_db`| host to install postgreSQL |

Role Variables
--------------

#### Installation options

| Variable | Description | Default |
|:---------|:------------|:--------|
|`sso_db_install`| Do we install single postgresql DB using this role | `false` |
|`sso_realm`| Realm for client auth | |
|`sso_admin_password`| Temporary admin password, must be 12 caracters minimum | `remembertochangeme`|
|`sso_config_key_store_password`| keytore password | |
|`sso_rhbk_version`| Version of RHBK to install | `26.2.5`|
|`sso_download_url`| URL to download rhbk zip for air-gapped installation | `https://satellite.{{ ansible_domain }}/pub/rhbk-{{ rhbk_version }}.zip`

#### Database options

| Variable | Description | Default |
|:---------|:------------|:--------|
|`sso_db_user`| User allowed to connect to  keycloak database | |
|`sso_db_pass`| Password for keycloak database | |
|`sso_db_name`| Database name | |
|`sso_db_host`| Ip or fqdn of database server | |

Dependencies
------------

To install all the dependencies via galaxy:

    ansible-galaxy collection install -r meta/requirements.yml

Example Playbook
----------------

```yaml
---
- name: Install and configure postgresql for RHBK
  hosts: sso_db
  gather_facts: false
  tags: db

  vars:
    sso_db_install: false

  pre_tasks:
    # always gather facts
    - setup:
      tags: always

  tasks:
    - name: Configure postgreSQL from sso role
      ansible.builtin.include_role:
        name: rhbk_poc
        tasks_from: postgresql.yaml
      when: sso_db_install | bool

- name: Install and configure RHBK
  hosts: sso_servers
  gather_facts: false
  become: true
  tags: rhbk

  pre_tasks:
    # always gather facts
    - setup:
      tags: always

  roles:
    - rhbk_poc

  vars:
    sso_db_user: "keycloak"
    sso_db_pass: "keycloak"
    sso_db_name: "keycloak"
    sso_db_host: "db.my_domain.fr"
    sso_realm: "MY_REALM"
    sso_admin_password: "remembertochangeme"
    sso_config_key_store_password: "MyIncr3dibleS3cur3P4ssw0rd" # notsecret

- name: Install HAproxy
  hosts: sso_haproxy
  gather_facts: false
  become: true
  tags: haproxy

  pre_tasks:
    # always gather facts
    - setup:
      tags: always

  tasks:
    - name: Configure HAproxy from sso roles
      ansible.builtin.include_role:
        name: rhbk_poc
        tasks_from: haproxy.yaml

  vars:
    sso_realm: "MY_REALM"
```

License
-------

BSD

Author Information
------------------

[Stephane V.](https://www.gnali.org) : tinsjourney@mastodon.top
