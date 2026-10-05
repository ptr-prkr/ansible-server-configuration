# Ansible Server Configuration

A practical configuration-management project demonstrating how Ansible can configure and maintain a Linux-based DevOps server in a repeatable and idempotent way.

The project uses a role-based Ansible architecture to manage common Linux configuration, Docker resources and an Nginx web server. It demonstrates inventories, variables, gathered facts, Jinja2 templates, handlers and privilege escalation.

## Project Overview

The objective of this project is to replace manual server configuration with declarative, repeatable infrastructure automation.

Ansible manages an Ubuntu server and ensures that:

- common administration packages are installed;
- a standard application directory and managed information file exist;
- Docker is available and running;
- Docker is enabled at system startup;
- a dedicated Docker network exists;
- Nginx is installed, running and enabled;
- an Ansible-managed web root exists;
- an HTML page is generated from a Jinja2 template;
- an Nginx virtual host is generated from configuration variables;
- the Nginx site is enabled;
- Nginx reloads only when its configuration changes.

The resulting configuration can be safely applied repeatedly without making unnecessary changes.

## Architecture

```text
                    Ansible Controller
                         node01
                            |
                            v
                     site.yml playbook
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
        common role     docker role     nginx role
             |              |              |
             |              |              +--> Jinja2 templates
             |              |              +--> Nginx configuration
             |              |              +--> Web content
             |              |              +--> Reload handler
             |              |
             |              +--> Docker service
             |              +--> Docker network
             |
             +--> Linux packages
             +--> Managed directories/files
```

The inventory is deliberately structured using a `devops_servers` group so additional Linux hosts can be introduced later without redesigning the playbook.

## Technologies

- Ansible Core
- YAML
- Jinja2
- Ubuntu Linux
- Docker
- Nginx
- Git
- SSH
- Python

## Ansible Concepts Demonstrated

### Inventory

The project uses a YAML inventory to define managed hosts and organise them into the `devops_servers` group.

### Playbooks

`site.yml` acts as the main entry point and applies the required roles to managed servers.

### Roles

Configuration is separated into reusable responsibilities:

- `common` — base Linux packages, directories and managed files
- `docker` — Docker service validation and network management
- `nginx` — Nginx service, web content and configuration

This avoids creating a single large playbook and makes the configuration easier to maintain and extend.

### Variables

Environment-specific configuration is separated from the role implementation.

For example:

```yaml
nginx_listen_port: 80
nginx_site_name: ansible-managed
nginx_document_root: /var/www/ansible-managed
```

These values are consumed by the Nginx role and its templates.

### Jinja2 Templates

Ansible dynamically generates both the web page and Nginx configuration.

Gathered Ansible facts are used to insert information such as:

- hostname;
- Linux distribution;
- operating-system version;
- system architecture.

### Handlers

The Nginx configuration tasks notify a dedicated reload handler.

Nginx is therefore reloaded only when its configuration actually changes rather than on every playbook execution.

### Idempotency

A key objective of the project is idempotent configuration management.

When a resource does not match the declared state, Ansible changes it. Once the desired state has been reached, subsequent executions make no unnecessary changes.

A successful repeated execution produced:

```text
ok=14
changed=0
unreachable=0
failed=0
```

## Project Structure

```text
ansible-server-configuration/
├── group_vars/
│   └── all.yml
├── inventory/
│   └── hosts.yml
├── roles/
│   ├── common/
│   │   └── tasks/
│   │       └── main.yml
│   ├── docker/
│   │   └── tasks/
│   │       └── main.yml
│   └── nginx/
│       ├── handlers/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           ├── ansible-managed.conf.j2
│           └── index.html.j2
├── screenshots/
├── ansible.cfg
├── site.yml
└── README.md
```

## Running the Playbook

Check the inventory:

```bash
ansible-inventory --graph
```

Validate the playbook:

```bash
ansible-playbook site.yml --syntax-check
```

Run the configuration:

```bash
ansible-playbook site.yml --ask-become-pass
```

Ansible requests the sudo password for tasks requiring privilege escalation.

## Validation

The completed configuration was independently verified by checking:

- Ansible playbook syntax;
- inventory structure;
- managed Linux resources;
- Docker network configuration;
- Nginx service state;
- HTTP availability;
- custom Ansible-managed HTTP response header.

The managed Nginx service returns:

```text
HTTP/1.1 200 OK
X-Managed-By: Ansible
```

## Screenshots

### Operational Validation

![Operational validation](screenshots/01%20operational%20validation.png)

Validation of the Ansible inventory, managed Linux server, Docker network and Nginx HTTP endpoint.

### Idempotent Execution

![Ansible idempotency](screenshots/02%20idempotency%20%28changed%3D0%29.png)

A complete playbook execution after the desired state has already been reached. All fourteen tasks complete successfully with `changed=0` and `failed=0`.

### Role-Based Ansible Architecture

![Ansible role architecture](screenshots/03%20professional%20Ansible%20code%20role%20architecture.png)

VS Code Remote SSH view of the role-based repository structure and the Nginx configuration-management tasks.

## Future Extension

The inventory and role structure are designed so the same configuration can later be applied to additional Linux hosts.

A natural extension is to combine this project with Terraform:

```text
Terraform
    |
    v
Provision infrastructure
    |
    v
Ansible
    |
    v
Configure operating system
    |
    +--> Docker
    |
    +--> Nginx
    |
    v
Application workload
```

This separates infrastructure provisioning from operating-system and application configuration.

## Key Outcome

This project demonstrates configuration management as code rather than a collection of manual server-administration commands.

The same declared configuration can be applied repeatedly, with Ansible changing only resources that differ from the required state.