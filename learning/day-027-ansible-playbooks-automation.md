# Day 27 — Automation with Ansible Playbooks

## Today's objective

Session 27 builds on the Ansible foundations from Day 26. You'll organize playbooks with tags, use Jinja2 templates, protect secrets with Ansible Vault, trigger handlers with notifications, and automate repeatable software configuration across multiple hosts.

By the end, you'll have a small Nginx configuration playbook that demonstrates templates, handlers, tags, and secret handling.

## 7-hour study plan

| Time | Topic | Deliverable |
|---|---|---|
| Hour 1 | Review inventory, playbooks, and idempotency | Working inventory and connectivity check |
| Hour 2 | Tags and selective execution | Run only chosen task groups |
| Hour 3 | Jinja2 templates and variables | Render a host-specific web page |
| Hour 4 | Handlers and notifications | Restart/reload only when configuration changes |
| Hour 5 | Ansible Vault | Encrypt a practice secret file |
| Hour 6 | Multi-host execution and troubleshooting | Validate the playbook on lab hosts |
| Hour 7 | Documentation, GitHub, and interview practice | Commit the lesson and verified results |

## 1. Project layout

Create a dedicated directory:

```bash
mkdir -p ansible/day-27/templates ansible/day-27/group_vars
cd ansible/day-27
```

Use a layout like this:

```text
ansible/day-27/
├── ansible.cfg
├── inventory.ini
├── site.yml
├── templates/
│   └── index.html.j2
└── group_vars/
    └── web.yml
```

Reuse the inventory from Day 26, but replace documentation-only example addresses with your actual lab hosts. Only target machines you own or are authorized to administer.

Example inventory:

```ini
[web]
web1 ansible_host=YOUR_HOST_1 ansible_user=ubuntu
# Add another reachable host if available:
# web2 ansible_host=YOUR_HOST_2 ansible_user=ubuntu
```

Optional project-local `ansible.cfg`:

```ini
[defaults]
inventory = ./inventory.ini
host_key_checking = True
retry_files_enabled = False
```

Host-key checking is intentionally enabled; verify host fingerprints through a trusted channel rather than disabling this protection to silence SSH warnings.

## 2. Tags: run selected parts of a playbook

Tags let you select tasks for focused runs, such as deploying an application or validating configuration. Add tags to tasks:

```yaml
tasks:
  - name: Install Nginx
    ansible.builtin.package:
      name: nginx
      state: present
    become: true
    tags:
      - install

  - name: Deploy web page
    ansible.builtin.template:
      src: templates/index.html.j2
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: "0644"
    become: true
    tags:
      - configure
      - deploy
```

Useful commands:

```bash
ansible-playbook site.yml --list-tags
ansible-playbook site.yml --list-tasks
ansible-playbook site.yml --tags configure --check --diff
ansible-playbook site.yml --tags install
```

Be careful: selecting tags can skip prerequisite tasks. Review the task list and target hosts before applying a partial run.

## 3. Jinja2 templates

Jinja2 lets Ansible render files using variables and facts. Create `group_vars/web.yml`:

```yaml
---
site_title: "DevOps Lab"
site_message: "Configured with Ansible"
```

Create `templates/index.html.j2`:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>{{ site_title }}</title>
</head>
<body>
  <h1>{{ site_title }}</h1>
  <p>{{ site_message }}</p>
  <p>Managed host: {{ inventory_hostname }}</p>
  <p>Operating system: {{ ansible_distribution | default('unknown') }}</p>
</body>
</html>
```

Ansible substitutes values inside `{{ ... }}`. The `default` filter prevents an undefined OS fact from breaking this simple template if facts are unavailable.

## 4. Handlers and notifications

Handlers run when notified by a task that reports a change. This is useful for restarting or reloading services only when their configuration changes.

Create `site.yml`:

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  gather_facts: true

  vars:
    web_package: nginx
    web_service: nginx

  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: "{{ web_package }}"
        state: present
      tags: [install]

    - name: Deploy the web page from a Jinja2 template
      ansible.builtin.template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: "0644"
      notify: Reload web service
      tags: [configure, deploy]

    - name: Ensure web service is enabled and running
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: started
        enabled: true
      tags: [service]

  handlers:
    - name: Reload web service
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: reloaded
```

**Important:** Not all services support reload. If Nginx or another target service does not support the requested operation on your platform, use a supported restart operation or validate the service-specific behavior first. A handler is only triggered when its notifying task reports a change.

Validate and preview:

```bash
ansible-playbook site.yml --syntax-check
ansible-playbook site.yml --list-tasks
ansible-playbook site.yml --check --diff
```

If the preview is safe and the host is your lab target, apply:

```bash
ansible-playbook site.yml
```

Run it again and observe the task status. If the rendered file is unchanged, the template task should normally report `ok` and the handler should not run.

## 5. Ansible Vault: protect secrets

Never store plaintext passwords, tokens, or private keys in a public repository. Ansible Vault can encrypt files or individual values.

Create a small local secrets file:

```bash
mkdir -p vault
printf '%s\n' 'demo_api_token: replace-with-a-lab-only-value' > vault/secrets.yml
ansible-vault encrypt vault/secrets.yml
```

You'll be prompted to set a Vault password. Do not use a real production secret for this practice task.

Other useful commands:

```bash
ansible-vault view vault/secrets.yml
ansible-vault edit vault/secrets.yml
ansible-vault decrypt vault/secrets.yml
```

Avoid decrypting secrets into a tracked plaintext file. Keep the Vault password out of source control; use a secure prompt or approved secret manager in real workflows. Add any local password file to `.gitignore`, and don't commit it.

To load encrypted variables in a playbook, add:

```yaml
vars_files:
  - vault/secrets.yml
```

Then run with an interactive Vault prompt:

```bash
ansible-playbook site.yml --ask-vault-pass
```

Only reference a secret variable when a task genuinely needs it. Do not print secrets using debug tasks or include them in logs.

## 6. Multi-host automation

Ansible can apply the same playbook to every host in an inventory group. First inspect targets:

```bash
ansible web --list-hosts
ansible web -m ping
ansible-playbook site.yml --check --diff
```

For a gradual rollout, use a serial batch:

```yaml
- name: Configure web servers in batches
  hosts: web
  serial: 1
  become: true
  tasks:
    - name: Example connectivity validation
      ansible.builtin.ping:
```

This is an illustrative separate play. `serial: 1` limits each batch to one host; it does not replace testing, health checks, or a rollback plan. For production deployments, validate application health before moving to the next batch.

## 7. Troubleshooting

| Symptom | Checks |
|---|---|
| Template variable is undefined | Variable spelling, group_vars filename, inventory group, fact gathering |
| Handler does not run | Did the notifying task report `changed`? Is the handler name correct? |
| Service reload fails | Confirm service name and reload support; test service configuration |
| Vault password error | Correct password, encrypted file integrity, correct vault prompt |
| Tag run skips setup | Inspect `--list-tasks`; make sure dependencies aren't unintentionally skipped |
| Permission denied writing a file | Use appropriate privilege escalation and file ownership |
| Multi-host task fails on one server | Review per-host results, connectivity, OS differences, and package/service names |
| YAML syntax error | Indentation, colons, quoting, list structure |
| A task changes on every run | Compare rendered output; use state-aware modules and stable templates |

## 8. Interview questions

1. What are Ansible tags, and when would you use them?
2. What is Jinja2, and how does Ansible use templates?
3. What is the difference between a task and a handler?
4. When does a handler run?
5. How does Ansible Vault protect secrets?
6. Why should Vault passwords not be committed to Git?
7. How would you deploy changes to multiple servers safely?
8. What does `serial` do in a play?
9. How do you troubleshoot an undefined template variable?
10. How do idempotency and handlers reduce unnecessary service disruption?

## 9. Completion checklist

- [ ] Build the Day 27 directory and inventory.
- [ ] List playbook tasks and tags before execution.
- [ ] Create a Jinja2 template using variables and host facts.
- [ ] Deploy the template with the Ansible template module.
- [ ] Configure a handler that reacts to changes.
- [ ] Encrypt a lab-only secrets file with Ansible Vault.
- [ ] Run syntax checks and a check-mode preview.
- [ ] Test against a reachable lab host or hosts only.
- [ ] Verify the second run makes no unnecessary changes.
- [ ] Commit documentation without secrets, private keys, or Vault passwords.

**Next:** Day 28 — Kubernetes Introduction: orchestration, cluster architecture, Pods, Services, kubectl, and Minikube.
