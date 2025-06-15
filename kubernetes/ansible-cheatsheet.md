# Ansible Cheatsheet

Ansible is an open-source automation tool for configuration management, application
deployment, and task automation using simple YAML-based playbooks.

## Installation

### macOS

```bash
# Install via Homebrew
brew install ansible

# Install via pip
pip3 install ansible

# Install specific version
pip3 install ansible==6.0.0
```

### Linux (Ubuntu/Debian)

```bash
# Install via package manager
sudo apt update
sudo apt install ansible

# Install via pip
sudo apt install python3-pip
pip3 install ansible

# Add PPA for latest version
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

### CentOS/RHEL

```bash
# Install via yum/dnf
sudo yum install epel-release
sudo yum install ansible

# Or with dnf
sudo dnf install ansible
```

## Configuration

### Ansible Configuration File (ansible.cfg)

```ini
[defaults]
# Inventory file location
inventory = ./hosts

# Default remote user
remote_user = ansible

# SSH key file
private_key_file = ~/.ssh/id_rsa

# Disable host key checking
host_key_checking = False

# Default module path
library = /usr/share/ansible

# Gathering facts
gathering = smart
fact_caching = memory

# Logging
log_path = /var/log/ansible.log

# Callback plugins
callback_plugins = /usr/share/ansible/plugins/callback

# Roles path
roles_path = ./roles

[ssh_connection]
# SSH timeout
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

## Inventory

### Basic Inventory File (hosts)

```ini
# Simple host list
server1.example.com
server2.example.com
192.168.1.100

# Groups
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

# Groups of groups
[production:children]
webservers
databases

# Variables
[webservers:vars]
http_port=80
maxRequestsPerChild=808

[databases:vars]
db_port=3306
```

### YAML Inventory

```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        http_port: 80
        max_requests: 808
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
      vars:
        db_port: 3306
    production:
      children:
        webservers:
        databases:
```

### Dynamic Inventory

```bash
# Use dynamic inventory script
ansible-playbook -i inventory_script.py playbook.yml

# AWS EC2 dynamic inventory
ansible-playbook -i aws_ec2.yml playbook.yml
```

## Basic Commands

### ansible Command

```bash
# Basic syntax
ansible <host-pattern> -m <module> -a "<module-args>"

# Ping all hosts
ansible all -m ping

# Ping specific group
ansible webservers -m ping

# Run command on all hosts
ansible all -m command -a "uptime"
ansible all -a "uptime"  # command module is default

# Run with sudo
ansible all -m command -a "whoami" --become
ansible all -a "whoami" -b  # Short form

# Use specific user
ansible all -m ping -u ubuntu

# Use SSH key
ansible all -m ping --private-key ~/.ssh/my-key.pem

# Limit to specific hosts
ansible webservers -m ping --limit "web1.example.com"

# Run in parallel (default is 5)
ansible all -m ping -f 10
```

### ansible-playbook Command

```bash
# Run playbook
ansible-playbook playbook.yml

# Run with specific inventory
ansible-playbook -i hosts playbook.yml

# Check syntax
ansible-playbook --syntax-check playbook.yml

# Dry run (check mode)
ansible-playbook --check playbook.yml

# Run with specific tags
ansible-playbook --tags "configuration" playbook.yml

# Skip specific tags
ansible-playbook --skip-tags "deployment" playbook.yml

# Limit to specific hosts
ansible-playbook --limit "webservers" playbook.yml

# Start at specific task
ansible-playbook --start-at-task="Install packages" playbook.yml

# Verbose output
ansible-playbook -v playbook.yml   # verbose
ansible-playbook -vv playbook.yml  # more verbose
ansible-playbook -vvv playbook.yml # debug

# Use vault for encrypted files
ansible-playbook --ask-vault-pass playbook.yml
ansible-playbook --vault-password-file vault_pass.txt playbook.yml
```

## Common Modules

### System Modules

```bash
# Command module
ansible all -m command -a "ls -la /tmp"

# Shell module (supports pipes, redirects)
ansible all -m shell -a "ps aux | grep nginx"

# Script module
ansible all -m script -a "./my_script.sh"

# Raw module (no Python required)
ansible all -m raw -a "uptime"
```

### File Operations

```bash
# Copy files
ansible all -m copy -a "src=/local/file dest=/remote/file"

# Fetch files from remote
ansible all -m fetch -a "src=/remote/file dest=/local/path"

# File module (create, delete, permissions)
ansible all -m file -a "path=/tmp/test state=touch"
ansible all -m file -a "path=/tmp/test state=absent"
ansible all -m file -a "path=/tmp/dir state=directory mode=0755"

# Template module
ansible all -m template -a "src=config.j2 dest=/etc/config.conf"

# Lineinfile module
ansible all -m lineinfile -a "path=/etc/hosts line='192.168.1.1 server1'"
```

### Package Management

```bash
# APT (Debian/Ubuntu)
ansible all -m apt -a "name=nginx state=present"
ansible all -m apt -a "name=nginx state=latest"
ansible all -m apt -a "name=nginx state=absent"
ansible all -m apt -a "update_cache=yes upgrade=dist"

# YUM (CentOS/RHEL)
ansible all -m yum -a "name=httpd state=present"
ansible all -m yum -a "name=httpd state=latest"

# Package module (generic)
ansible all -m package -a "name=vim state=present"

# Pip module
ansible all -m pip -a "name=django state=present"
```

### Service Management

```bash
# Service module
ansible all -m service -a "name=nginx state=started"
ansible all -m service -a "name=nginx state=stopped"
ansible all -m service -a "name=nginx state=restarted"
ansible all -m service -a "name=nginx enabled=yes"

# Systemd module
ansible all -m systemd -a "name=nginx state=started daemon_reload=yes"
```

### User Management

```bash
# User module
ansible all -m user -a "name=john state=present"
ansible all -m user -a "name=john password={{ 'password' | password_hash('sha512') }}"
ansible all -m user -a "name=john groups=sudo append=yes"
ansible all -m user -a "name=john state=absent remove=yes"

# Group module
ansible all -m group -a "name=developers state=present"
```

## Playbooks

### Basic Playbook Structure

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    max_clients: 200
  
  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present
    
    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy configuration file
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
  
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Multi-Play Playbook

```yaml
---
- name: Configure database servers
  hosts: databases
  become: yes
  tasks:
    - name: Install MySQL
      package:
        name: mysql-server
        state: present

- name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install Apache
      package:
        name: apache2
        state: present
```

### Variables and Facts

```yaml
---
- name: Variable examples
  hosts: all
  vars:
    packages:
      - vim
      - git
      - curl
    user_info:
      name: john
      shell: /bin/bash
  
  tasks:
    - name: Install packages
      package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
    
    - name: Display system facts
      debug:
        msg: "System {{ ansible_hostname }} has {{ ansible_processor_cores }} cores"
    
    - name: Create user
      user:
        name: "{{ user_info.name }}"
        shell: "{{ user_info.shell }}"
```

### Conditionals

```yaml
---
- name: Conditional examples
  hosts: all
  tasks:
    - name: Install Apache on Ubuntu
      package:
        name: apache2
        state: present
      when: ansible_distribution == "Ubuntu"
    
    - name: Install Apache on CentOS
      package:
        name: httpd
        state: present
      when: ansible_distribution == "CentOS"
    
    - name: Restart service if config changed
      service:
        name: nginx
        state: restarted
      when: config_changed is defined and config_changed.changed
```

### Loops

```yaml
---
- name: Loop examples
  hosts: all
  tasks:
    - name: Install multiple packages
      package:
        name: "{{ item }}"
        state: present
      loop:
        - vim
        - git
        - curl
    
    - name: Create multiple users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
      loop:
        - { name: 'alice', groups: 'developers' }
        - { name: 'bob', groups: 'admins' }
    
    - name: Loop with dictionary
      debug:
        msg: "{{ item.key }} = {{ item.value }}"
      loop: "{{ my_dict | dict2items }}"
      vars:
        my_dict:
          key1: value1
          key2: value2
```

### Error Handling

```yaml
---
- name: Error handling examples
  hosts: all
  tasks:
    - name: Task that might fail
      command: /bin/false
      ignore_errors: yes
    
    - name: Task with rescue
      block:
        - name: Risky task
          command: /bin/false
      rescue:
        - name: Handle error
          debug:
            msg: "Task failed, handling error"
      always:
        - name: Always run
          debug:
            msg: "This always runs"
    
    - name: Fail with custom message
      fail:
        msg: "This is a custom failure message"
      when: some_condition is defined
```

## Jinja2 Templates

### Basic Template (nginx.conf.j2)

```jinja2
server {
    listen {{ http_port }};
    server_name {{ ansible_fqdn }};
    
    root /var/www/html;
    index index.html index.htm;
    
    # Generated on {{ ansible_date_time.iso8601 }}
    # System: {{ ansible_distribution }} {{ ansible_distribution_version }}
    
    {% if ssl_enabled %}
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    {% endif %}
    
    {% for location in custom_locations %}
    location {{ location.path }} {
        {{ location.config }}
    }
    {% endfor %}
}
```

### Template with Filters

```jinja2
# Uppercase filter
server_name {{ ansible_hostname | upper }};

# Default value
port {{ custom_port | default('80') }};

# List operations
{% for server in web_servers | sort %}
upstream_server {{ server }};
{% endfor %}

# Math operations
worker_processes {{ ansible_processor_cores | int * 2 }};

# String manipulation
log_file /var/log/{{ app_name | lower }}.log;
```

## Roles

### Role Directory Structure

```
roles/
  common/
    tasks/
      main.yml
    handlers/
      main.yml
    templates/
      ntp.conf.j2
    files/
      bar.txt
    vars/
      main.yml
    defaults/
      main.yml
    meta/
      main.yml
```

### Using Roles in Playbooks

```yaml
---
- name: Configure servers
  hosts: all
  roles:
    - common
    - { role: nginx, http_port: 80 }
    - role: mysql
      mysql_root_password: secret
      when: "'databases' in group_names"
```

### Role Dependencies (meta/main.yml)

```yaml
---
dependencies:
  - { role: common, some_parameter: 3 }
  - role: apache
    port: 80
  - role: postgres
    dbname: blarg
    other_parameter: 12
```

## Ansible Vault

### Vault Commands

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Rekey (change password)
ansible-vault rekey secrets.yml

# Encrypt string
ansible-vault encrypt_string 'secret_password' --name 'db_password'
```

### Using Vault in Playbooks

```yaml
---
- name: Deploy application
  hosts: webservers
  vars_files:
    - secrets.yml
  tasks:
    - name: Configure database
      mysql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"
        state: present
```

### Running with Vault

```bash
# Prompt for vault password
ansible-playbook --ask-vault-pass playbook.yml

# Use password file
ansible-playbook --vault-password-file vault_pass.txt playbook.yml

# Use environment variable
export ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt
ansible-playbook playbook.yml
```

## Advanced Features

### Include and Import

```yaml
---
- name: Main playbook
  hosts: all
  tasks:
    # Static include (processed at parse time)
    - import_tasks: common_tasks.yml
    
    # Dynamic include (processed at runtime)
    - include_tasks: "{{ ansible_distribution }}_tasks.yml"
    
    # Include with variables
    - include_tasks: database_tasks.yml
      vars:
        db_name: production
    
    # Conditional include
    - include_tasks: ssl_tasks.yml
      when: ssl_enabled | default(false)
```

### Custom Facts

```bash
# Create custom fact script
#!/bin/bash
echo '{"custom_fact": "custom_value"}'

# Place in /etc/ansible/facts.d/custom.fact
# Access as ansible_local.custom.custom_fact
```

### Delegation and Local Actions

```yaml
---
- name: Delegation examples
  hosts: webservers
  tasks:
    # Run on localhost
    - name: Run command locally
      command: echo "Running on control machine"
      delegate_to: localhost
    
    # Local action (shorthand)
    - name: Local action
      local_action: command echo "Local command"
    
    # Run once for all hosts
    - name: Run once
      command: echo "This runs once"
      run_once: true
    
    # Delegate facts gathering
    - name: Gather facts from another host
      setup:
      delegate_to: "{{ item }}"
      delegate_facts: true
      loop: "{{ groups['databases'] }}"
```

## Testing and Debugging

### Debug Module

```yaml
---
- name: Debug examples
  hosts: all
  tasks:
    - name: Debug variable
      debug:
        var: ansible_hostname
    
    - name: Debug with message
      debug:
        msg: "The hostname is {{ ansible_hostname }}"
    
    - name: Debug complex data
      debug:
        var: hostvars[inventory_hostname]
    
    - name: Conditional debug
      debug:
        msg: "This is a web server"
      when: "'webservers' in group_names"
```

### Assertion Module

```yaml
---
- name: Assert examples
  hosts: all
  tasks:
    - name: Assert condition
      assert:
        that:
          - ansible_os_family == "Debian"
          - ansible_python_version.startswith('3')
        fail_msg: "System requirements not met"
        success_msg: "System requirements satisfied"
```

## Common Patterns

### Rolling Updates

```yaml
---
- name: Rolling update
  hosts: webservers
  serial: 1  # Update one host at a time
  max_fail_percentage: 25
  
  pre_tasks:
    - name: Remove from load balancer
      uri:
        url: "http://lb.example.com/remove/{{ inventory_hostname }}"
        method: POST
  
  tasks:
    - name: Update application
      copy:
        src: app.tar.gz
        dest: /tmp/app.tar.gz
    
    - name: Restart service
      service:
        name: myapp
        state: restarted
  
  post_tasks:
    - name: Add back to load balancer
      uri:
        url: "http://lb.example.com/add/{{ inventory_hostname }}"
        method: POST
```

### Blue-Green Deployment

```yaml
---
- name: Blue-green deployment
  hosts: webservers
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') }}"
    deploy_color: "{{ 'green' if current_color == 'blue' else 'blue' }}"
  
  tasks:
    - name: Deploy to inactive environment
      copy:
        src: "app-{{ app_version }}.tar.gz"
        dest: "/opt/{{ deploy_color }}/app.tar.gz"
    
    - name: Update load balancer
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      vars:
        active_color: "{{ deploy_color }}"
      notify: reload nginx
```

## Performance and Best Practices

### Optimization Tips

```yaml
---
- name: Performance optimizations
  hosts: all
  gather_facts: no  # Skip fact gathering if not needed
  strategy: free    # Don't wait for all hosts
  
  tasks:
    - name: Gather minimal facts
      setup:
        gather_subset: min
      when: facts_needed | default(false)
    
    - name: Use changed_when
      command: some_command
      changed_when: false  # Don't report as changed
    
    - name: Use creates parameter
      command: create_file.sh
      args:
        creates: /path/to/file  # Skip if file exists
```

### Directory Layout

```
site.yml                 # master playbook
webservers.yml          # playbook for webserver tier
dbservers.yml           # playbook for dbserver tier

production/             # inventory file for production servers
staging/                # inventory file for staging environment

group_vars/
   group1.yml           # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml        # here we assign variables to particular systems
   hostname2.yml

library/                # if any custom modules, put them here (optional)
module_utils/           # if any custom module_utils to support modules, put them here (optional)
filter_plugins/         # if any custom filter plugins, put them here (optional)

roles/
    common/             # this hierarchy represents a "role"
        tasks/          #
            main.yml    #  <-- tasks file can include smaller files if warranted
        handlers/       #
            main.yml    #  <-- handlers file
        templates/      #  <-- files for use with the template resource
            ntp.conf.j2 #  <------- templates end in .j2
        files/          #
            bar.txt     #  <-- files for use with the copy resource
            foo.sh      #  <-- script files for use with the script resource
        vars/           #
            main.yml    #  <-- variables associated with this role
        defaults/       #
            main.yml    #  <-- default lower priority variables for this role
        meta/           #
            main.yml    #  <-- role dependencies
        library/        # roles can also include custom modules
        module_utils/   # roles can also include custom module_utils
        lookup_plugins/ # or other types of plugins, like lookup in this case

    webtier/            # same kind of structure as "common" was above, done for the webtier role
    monitoring/         # ""
    fooapp/             # ""
```

## Useful Commands Reference

| Command | Description |
|---------|-------------|
| `ansible --version` | Show Ansible version |
| `ansible all -m ping` | Ping all hosts |
| `ansible-doc <module>` | Show module documentation |
| `ansible-galaxy install <role>` | Install role from Galaxy |
| `ansible-playbook --syntax-check playbook.yml` | Check syntax |
| `ansible-playbook --check playbook.yml` | Dry run |
| `ansible-playbook --diff playbook.yml` | Show differences |
| `ansible-inventory --list` | List inventory |
| `ansible-config dump` | Show configuration |
| `ansible-vault --help` | Vault help |

## Environment Variables

```bash
# Common Ansible environment variables
export ANSIBLE_CONFIG=./ansible.cfg
export ANSIBLE_INVENTORY=./hosts
export ANSIBLE_REMOTE_USER=ansible
export ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/id_rsa
export ANSIBLE_HOST_KEY_CHECKING=False
export ANSIBLE_VAULT_PASSWORD_FILE=./vault_pass.txt
export ANSIBLE_ROLES_PATH=./roles
export ANSIBLE_LOG_PATH=./ansible.log
```

---

*For more detailed information, visit the [official Ansible documentation](https://docs.ansible.com/)*
