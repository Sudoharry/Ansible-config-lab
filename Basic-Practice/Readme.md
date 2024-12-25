# Basic Ansible Playbooks for DevOps Interviews

This repository contains a list of **basic Ansible playbooks** commonly discussed in **DevOps interviews**. These examples cover real-world use cases and essential concepts like managing files, services, users, packages, and troubleshooting. These playbooks will help you prepare for DevOps roles and demonstrate your understanding of Ansible and automation.

## List of Playbooks

### 1. **Install a Package (e.g., Nginx)**

### This playbook installs the **Nginx** package on the target hosts.

```yaml
---
- name: Install Nginx
  hosts: all
  become: true
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

### 2. **Start and Enable a Service**

### This playbook ensures that the Nginx service is both started and enabled to start at boot time.

```yaml
---
- name: Start and Enable Nginx
  hosts: all
  become: true
  tasks:
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: true

### 3. **Create a User with a Password**

### This playbook creates a new user and sets a hashed password for it.

```yaml
---
- name: Create a new user
  hosts: all
  become: true
  vars:
    user_password: "{{ 'mypassword' | password_hash('sha512') }}"
  tasks:
    - name: Create a user with a hashed password
      user:
        name: devops_user
        password: "{{ user_password }}"
        state: present


### 4. **Copy a File to Remote Hosts**

### This playbook copies a file from the local machine to the remote hosts.

```yaml
---
- name: Copy file to remote host
  hosts: all
  become: true
  tasks:
    - name: Copy a configuration file
      copy:
        src: /local/path/to/file.conf
        dest: /etc/nginx/conf.d/file.conf
        mode: '0644'

### 5. **Execute a Command on Remote Hosts**

### This playbook executes a command on remote hosts (e.g., uptime).

```yaml
---
- name: Run a command on remote hosts
  hosts: all
  tasks:
    - name: Execute uptime command
      command: uptime


### 6. **Create a Directory**

### This playbook ensures a specific directory exists on remote hosts.

```yaml
---
- name: Create a directory
  hosts: all
  become: true
  tasks:
    - name: Ensure a directory exists
      file:
        path: /var/log/devops
        state: directory
        mode: '0755'

### 7. **Set up a Cron Job**

### This playbook adds a cron job that runs a specific command at a scheduled time.

```yaml
---
- name: Set up a cron job
  hosts: all
  become: true
  tasks:
    - name: Add a cron job to clear temp files
      cron:
        name: "Clear temp files"
        minute: "0"
        hour: "0"
        job: "rm -rf /tmp/*"


### 8. **Gather Facts and Display Them**

### This playbook gathers system facts and displays them using the debug module.

```yaml
---
- name: Gather system facts
  hosts: all
  tasks:
    - name: Display system facts
      debug:
        var: ansible_facts

### 9. **Deploy an Application (e.g., Web Application)**

###This playbook installs Apache and deploys a simple web application.

```yaml
---
- name: Deploy a sample web application
  hosts: all
  become: true
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Copy application files
      copy:
        src: /local/path/to/index.html
        dest: /var/www/html/index.html

### 10. **Check Disk Space and Alert if Low**

### This playbook checks disk space usage and fails the playbook if usage exceeds a certain threshold.

```yaml
---
- name: Check disk space
  hosts: all
  tasks:
    - name: Check disk usage
      shell: df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
      register: disk_usage

    - name: Fail if disk usage is more than 80%
      fail:
        msg: "Disk usage is critically high: {{ disk_usage.stdout }}%"
      when: disk_usage.stdout | int > 80

###Why These Playbooks Are Important
 - Real-World Scenarios: These playbooks simulate real DevOps tasks like managing services, users, and system configuration.
 - Ansible Modules: They showcase your understanding of core Ansible modules such as apt, service, user, file, cron, and debug.
 - Automation: These playbooks highlight how Ansible can be used to automate routine system administration tasks.
 - Problem-Solving: Troubleshooting playbooks are essential to demonstrate your problem-solving skills in DevOps.

###Be Ready to Explain

- In an interview, be ready to discuss:

 1) Purpose of each playbook.
 2) Modules used in each playbook.
 3) Customization options and how you would modify the playbooks for different scenarios.
 4) Optimizations or improvements you would make to these playbooks.

###Conclusion
 - This collection of basic Ansible playbooks covers key tasks you'll encounter in a DevOps role. Understanding these playbooks will help you perform essential operations and showcase your skills in Ansible and automation during interviews.
