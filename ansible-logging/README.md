### Need to add loki health check
Custom Modules
Write a Python module to check Loki’s health.
### Parallel Execution
Use strategy: free to deploy Loki and Promtail at the same time.
```yaml
- name: Deploy Loki and Promtail
  hosts: kube_cluster
  strategy: free
  gather_facts: true

  tasks:
    - name: Deploy Loki
      ansible.builtin.include_role:
        name: loki
      when: loki_enabled

    - name: Deploy Promtail
      ansible.builtin.include_role:
        name: promtail
      when: promtail_enabled
```
### Asynchronous Tasks (Background Execution)
Deploy Promtail without blocking execution.
```yaml
- name: Install Promtail asynchronously
  kubernetes.core.helm:
    name: promtail
    chart_ref: grafana/promtail
    release_namespace: "{{ promtail_namespace }}"
    create_namespace: true
    values_files:
      - /tmp/promtail-values.yaml
    state: present
  async: 300
  poll: 0

- name: Check Promtail deployment status
  async_status:
    jid: "{{ ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 10
  delay: 30
```
