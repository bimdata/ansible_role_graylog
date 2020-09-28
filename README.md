Ansible role graylog
=========

This role installs and configures [Graylog](https://www.graylog.org/).

This role assumes you have others roles for Elasticsearch, MongoDB, Nginx etc.

Requirements
------------

* You need a running Elasticsearch cluster and a MongoDB cluster.
* Managed servers must be able to communicate with https://packages.graylog2.org/.
* This role is only tested on Debian 10.x (Buster).

Role Variables
--------------

This role tries to keep the same default configuration as if you manually install
Graylog. All default values are defined in `./defaults/main.yml`, you may want to
check it.
We try to keep the same name for the Ansible variable as the name in the Graylog
configuration file, but with the prefix `graylog_`. You can have more information
about each parameter in the [Graylog documentation](https://docs.graylog.org/en/latest/pages/configuration/server.conf.html).

You need to define at least two variables :
* `graylog_password_secret`: you should generate its content with the command:
`pwgen -N 1 -s 96` or `tr -cd '[:alnum:]' < /dev/urandom | fold -w96 | head -n1`
* `graylog_root_password_sha2`: you should generate its content with the command:
`echo -n your_password | shasum -a 256` (You need to replace 'your_password' by
an actual password!)  
Or you can use Ansible `hash()` function:
`graylog_root_password_sha2: "{{ vault_graylog_root_password | hash('sha256') }}"`.


This role supports the installation of plugins too. Plugins need to be `.jar` files,
and the managed hosts must be able to download them. The checksum is optional and work
like the checksum parameter in [Ansible get_url module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html#parameter-checksum).
```
graylog_plugins:
  - url: https://github.com/graylog-labs/graylog-plugin-metrics-reporter/releases/download/3.0.0/metrics-reporter-prometheus-3.0.0.jar
    checksum: sha256:383eac2135baf248b5a0828a9e305130a2ab863b07afeef30cba00be05fc7cf9
```

If some of your plugins need configuration in the Graylog main configuration file, there is
a special variable for that `graylog_custom_config`. This is a dictionary, the keys are used
as the option names and the values as values. For example:
```
graylog_custom_config:
  metrics_prometheus_enabled: true
  metrics_prometheus_report_interval: 1m
  metrics_prometheus_address: 127.0.0.1:9001
  metrics_prometheus_job_name: graylog
```
will add at the end of `/etc/graylog/server/server.conf`:
```
# Custom configuration, if some are needed by plugins.
metrics_prometheus_enabled = True
metrics_prometheus_report_interval = 1m
metrics_prometheus_address = 127.0.0.1:9001
metrics_prometheus_job_name = graylog
```

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

```
- hosts: logs-servers
  gather_facts: True
  become: true
  vars:
    graylog_password_secret: "OMFPRQwk7Pg7i9Apun5xbuK4ICl0cfNUbZ5QblvmHKnKvnpzbjxtgHIoaSiEmi9XVlbqDhI6d8UqErW2wRiS0uapaHRgW4e"
    graylog_root_password_sha2: "4da3376323046a3bb6759f0a3f4ae7100a0567950c53ee42d2e19201baaa6dfc"
    # You can also ask Ansible to hash it and use a vault to store the password.
    # graylog_root_password_sha2: "{{ vault_graylog_root_password | hash('sha256') }}"
  roles:
    - role: bimdata.graylog
```

License
-------

MIT

Author Information
------------------

[BIMData.io](https://bimdata.io/)
