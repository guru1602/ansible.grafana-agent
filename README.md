Grafana Agent Ansible Role
=========

Install and configure grafana agent

Requirements
------------

* debian based distribution

Example Playbook
----------------

```yaml
---
- name: Playbook to monitor servers
  hosts: servers
  remote_user: ansible
  become: true
  vars:
    ansible_python_interpreter: /usr/bin/python3
  roles:
    - role: jindrichskupa.ansible_grafana_agent
      vars:
        # Agent version, default is 0.15.0
        grafana_agent_version: 0.15.0
        # Remote Write Endpoint (grafana cloud)
        grafana_cloud_url: https://prometheus-us-central1.grafana.net/api/prom/push
        # Username / Instance ID in prometheus details (grafana cloud)
        grafana_cloud_username: "1234567890"
        # Password / API Key in prometheus details (grafana cloud)
        grafana_cloud_password: password
        # optional, default/example will be used instead
        grafana_agent_template: grafana/grafana-agent.yaml
        # additional scrapes, default interval is 15s
        grafana_scrapes:
          - job_name: openvpn-users
            target: localhost:9176
```

Example Template
----------------

```yaml
integrations:
  node_exporter:
    enabled: true
  prometheus_remote_write:
  - basic_auth:
      password: {{ grafana_cloud_password }}
      username: {{ grafana_cloud_username }}
    url: {{ grafana_cloud_url }}
prometheus:
  configs:
  - name: integrations
    remote_write:
    - basic_auth:
        password: {{ grafana_cloud_password }}
        username: {{ grafana_cloud_username }}
      url: {{ grafana_cloud_url }}
    scrape_configs: {% if grafana_scrapes is not defined %}[]
{% else %}
{% for scrape in grafana_scrapes %}
    - job_name: {{ scrape.job_name }}
      static_configs:
      - targets:
        - {{ scrape.target }}
{% endfor %}
{% endif %}
  global:
    scrape_interval: 15s
  wal_directory: /tmp/grafana-agent-wal
server:
  http_listen_port: 12345
```

License
-------

MIT

Author Information
------------------

Jindrich Skupa
