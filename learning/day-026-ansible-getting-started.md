# Day 26 — Getting Started with Ansible

## Today's goal

Session 26 introduces configuration management, push vs pull models, Ansible architecture, inventories, ad-hoc commands, roles, collections, variables, facts, and an idempotent deployment.

By the end, you should be able to manage one or more Linux hosts over SSH, run safe ad-hoc commands, and write a small playbook that installs and starts Nginx without repeatedly changing a host that is already in the desired state.

## 7-hour study plan

| Time | Focus | Deliverable |
|---|---|---|
| Hour 1 | Configuration management and Ansible architecture | Explain control node, managed nodes, inventory, modules |
| Hour 2 | Installation and SSH connectivity | Ansible version and successful connectivity check |
| Hour 3 | Inventory and ad-hoc commands | Run ping, gather facts, and inspect disk space |
| Hour 4 | YAML playbooks and variables | A reusable Nginx playbook |
| Hour 5 | Idempotency, handlers, and roles | Understand changed vs ok and organize a role |
| Hour 6 | Troubleshooting and safe practice | Resolve a deliberate inventory/SSH issue |
| Hour 7 | GitHub documentation and interview practice | Commit notes and verified lab evidence |

## 1. What is configuration management?

Configuration management keeps servers in a known, repeatable desired state: required packages installed, services running, configuration files present, and settings consistent.

Ansible is agentless for typical Linux SSH management: it runs from a control node, connects to managed hosts (usually with SSH), executes modules, and reports results. Managed nodes generally need SSH access and Python for most POSIX modules.

### Push vs pull

- **Push model:** the controller initiates changes on managed hosts. Ansible commonly uses this model.
- **Pull model:** managed hosts periodically retrieve their desired configuration from a central service.

### Ansible architecture

```text
Control node
  ├── Inventory: which hosts?
  ├── Playbook: desired state?
  └── Modules: what actions?
          |
          | SSH
          v
Managed Linux host(s)
  ├── Packages
  ├── Configuration
  └── Services
```

## 2. Install Ansible and prepare access

Use a Linux control node or WSL/Linux VM where possible. Install Ansible using the official instructions for your distribution. Verify:

```bash
ansible --version
python3 --version
ssh -V
```

You need a reachable Linux managed host and an SSH account authorized to perform the required tasks. A local VM is suitable for practice. Don't expose SSH publicly just to complete this lab; use a private network, VPN, or tightly restricted source IP where possible.

Check SSH connectivity directly before using Ansible:

```bash
ssh -i ~/.ssh/your-lab-key user@YOUR_HOST
```

Replace the key path, username, and host with your own values. Never commit private keys or passwords.

## 3. Create an inventory

Create a project folder:

```bash
mkdir -p ansible/day-26
cd ansible/day-26
```

Create `inventory.ini`:

```ini
[web]
web1 ansible_host=192.0.2.10 ansible_user=ubuntu

[web:vars]
ansible_python_interpreter=auto_silent
```

The address `192.0.2.10` is a documentation-only example and will not reach a real server. Replace it with the address of your own lab host. Adjust the SSH username for your operating system.

Check which hosts Ansible sees:

```bash
ansible-inventory -i inventory.ini --list
ansible -i inventory.ini web --list-hosts
```

## 4. Run safe ad-hoc commands

Ad-hoc commands are useful for one-off tasks. Start with read-only checks:

```bash
ansible -i inventory.ini web -m ping
ansible -i inventory.ini web -m setup -a 'filter=ansible_distribution*'
ansible -i inventory.ini web -m command -a 'uptime'
ansible -i inventory.ini web -m command -a 'df -h'
```

The Ansible `ping` module checks that Ansible can connect and execute Python-based module code; it is not an ICMP network ping.

Avoid using broad ad-hoc commands such as `shell: rm -rf ...` on managed hosts. Prefer purpose-built modules and inspect targets before making changes.

## 5. Write an idempotent Nginx playbook

Create `webserver.yml`:

```yaml
---
- name: Configure a basic web server
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

    - name: Ensure Nginx is enabled and running
      ansible.builtin.service:
        name: "{{ web_service }}"
        state: started
        enabled: true
```

This playbook uses generic package/service modules. Check that Nginx's package and service names are correct for your managed operating system.

Validate the YAML/playbook and inspect the target list:

```bash
ansible-playbook -i inventory.ini webserver.yml --syntax-check
ansible-playbook -i inventory.ini webserver.yml --list-hosts
ansible-playbook -i inventory.ini webserver.yml --check --diff
```

Check mode is a useful preview, but not every module can predict every change. If the preview looks safe and the target is your lab host, run:

```bash
ansible-playbook -i inventory.ini webserver.yml
```

Run it a second time. When the host already matches the desired state, the package and service tasks should generally report `ok`, not `changed`. This is the central idea of **idempotency**: repeated runs converge on the desired state without unnecessary changes.

If you want to test the HTTP service and the firewall/network allow it, run from a machine that can reach the managed host:

```bash
curl -I http://YOUR_HOST
```

Do not assume public access is configured; routing, firewalls, security groups, and host policy determine reachability.

## 6. Variables, facts, roles, and collections

### Variables

Variables let you reuse a playbook with different package names, ports, or environment values. Keep non-secret configuration in version control; use Ansible Vault or a secret manager for sensitive values.

### Facts

Facts are system details gathered from managed hosts, such as OS family, IP addresses, memory, and processor information. You can inspect them with:

```bash
ansible -i inventory.ini web -m setup
```

Gather only the facts you need when optimizing larger playbooks.

### Roles

Roles organize tasks, handlers, variables, templates, and files into a reusable layout. A typical role structure is:

```text
roles/
└── webserver/
    ├── tasks/main.yml
    ├── handlers/main.yml
    ├── defaults/main.yml
    ├── templates/
    └── files/
```

The playbook can then call the role:

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - webserver
```

Create the role structure and move the Nginx tasks into `roles/webserver/tasks/main.yml` as an extension exercise.

### Collections

Collections package Ansible content such as modules, plugins, and roles. Discover installed collections with:

```bash
ansible-galaxy collection list
```

Use fully qualified module names such as `ansible.builtin.package` to make the module source clear.

## 7. Idempotency and change control

A well-written playbook describes the desired end state. Prefer modules that model state, such as `package: state=present` and `service: state=started`, over commands that always run.

- `ok`: no change was needed.
- `changed`: Ansible changed the host or reports a task as changed.
- `failed`: the task failed; inspect the message and target state.
- `unreachable`: Ansible could not connect to the host.

Use `--check --diff` for a preview where supported, target a specific inventory group, and keep privileged access minimal. Avoid logging secrets or using verbose output carelessly when sensitive values could appear.

## 8. Troubleshooting

| Symptom | What to check |
|---|---|
| Host is unreachable | IP/DNS, route, firewall, SSH service, port 22 access |
| Permission denied | SSH user, key path, key permissions, authorized key |
| sudo/become fails | Privilege escalation permissions and whether a password is required |
| Python/module error | Python installed on target and supported interpreter |
| Inventory is empty | Inventory path, INI syntax, group name |
| Package not found | Managed OS and package name |
| Service name is invalid | OS-specific service name and installed package |
| YAML parsing error | Indentation, colons, quotes, and list markers |
| Task changes every run | Check whether a command is being used instead of a state-aware module |
| Check mode differs from real run | Module check-mode support and external conditions |

## 9. Interview questions

1. What is configuration management?
2. How does Ansible's push model work?
3. What is the control node, and what is a managed node?
4. What is an inventory?
5. What does Ansible's ping module actually test?
6. What is idempotency, and why does it matter?
7. What is the difference between an ad-hoc command and a playbook?
8. What are facts, variables, roles, and collections?
9. How do `become: true` and SSH authentication differ?
10. How would you protect secrets in Ansible?

## 10. Completion checklist

- [ ] Install Ansible and verify its version.
- [ ] Confirm SSH access to a lab Linux host.
- [ ] Create an inventory and inspect its hosts.
- [ ] Run ping, setup, uptime, and disk-space ad-hoc checks.
- [ ] Write and syntax-check the Nginx playbook.
- [ ] Preview changes with check mode and diff.
- [ ] Run the playbook on a lab host, if available.
- [ ] Run it again and inspect idempotency.
- [ ] Understand facts, variables, roles, and collections.
- [ ] Document the lab without committing credentials or private keys.

**Next:** Day 27 — Automation with Ansible Playbooks: tags, Jinja2 templates, Ansible Vault, handlers/notifiers, and repeatable multi-node configuration.
