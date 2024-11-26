# Troubleshooting Guide for Automated Patch Management and Software Installation (Docker & Nginx)

This guide documents the troubleshooting steps and practices for resolving issues while running an Ansible playbook to manage Docker and Nginx installations on Ubuntu-based systems.

## Table of Contents

1. [Introduction](#introduction)
2. [Update all Packages](#update-all-packages)
3. [Gather Ubuntu Release Codename](#gather-ubuntu-release-codename)
4. [Add Docker GPG Key](#add-docker-gpg-key-to-trusted-sources)
5. [Add Docker Repository](#add-docker-repository-on-ubuntu-based-systems)
6. [Update APT Cache](#update-apt-cache)
7. [Install Required Software](#install-required-software-docker-and-nginx)
8. [Restart Services](#restart-services-if-necessary)
9. [Ensure Log Directory Exists](#ensure-log-directory-exists)
10. [Log Update Results](#log-update-results)
11. [Execution](#execution)
12. [Output](#output)
13. [License](#license)

---

## Introduction
-This Ansible playbook provides a robust solution for system administrators to automate patch management and software installation. 
- It simplifies routine tasks like updating packages, adding repositories, and logging updates for better troubleshooting and system auditing.

## Requirements
- Ansible 2.9+ installed on the control node.
- SSH access to the target nodes.
- Target nodes running Ubuntu (any supported version).

## Tasks Overview

### Update all Packages
   - Purpose: Ensures that all packages on the system are up-to-date.
       
   - Task: Uses the ansible.builtin.apt module to perform a system-wide package upgrade.
   - Code:
          - name: Update all packages to the latest version
          ansible.builtin.apt:
          upgrade: dist
          update_cache: yes
          state: latest
          register: update_result
          
   - Outcome: All packages are updated, and results are stored in the update_result variable.

### Gather Ubuntu Release Codename
 - Purpose: Fetches the release codename of the system (e.g., focal for Ubuntu 20.04).
       
 - Task: Sets a fact using ansible_facts.lsb.codename.
 - Code:
         - name: Gather the Ubuntu release codename
           ansible.builtin.set_fact:
             ubuntu_codename: "{{ ansible_facts.lsb.codename }}"
                
 - Outcome: The release codename is stored for later use in repository configurations.
### Add Docker GPG Key
  - Purpose: Ensures that the Docker repository's authenticity can be verified.
       
  - Task: Downloads the Docker GPG key and places it in the trusted sources directory.
  - Code:
       - ```name: Add Docker GPG key to trusted sources
           ansible.builtin.get_url:
           url: https://download.docker.com/linux/ubuntu/gpg
           dest: /etc/apt/trusted.gpg.d/docker.asc
              
  - Outcome: The Docker GPG key is downloaded and trusted by the system.

### Add Docker Repository
   - Purpose: Adds the Docker repository for installing Docker packages.
     
   - Task: Configures the repository URL and associates it with the downloaded GPG key.
   - Code:
       - ```name: Add Docker repository on Ubuntu-based systems
          ansible.builtin.apt_repository:
          repo: "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/docker.asc] https://download.docker.com/linux/ubuntu {{ 
          ubuntu_codename }} stable"
          state: present
          
   - Outcome: Docker repository is added and ready for package installation.

### Update APT Cache
   - Purpose: Refreshes the package database to include the newly added repository.
     
   - Task: Updates the APT cache using ansible.builtin.apt.
   - Code:
       - ```name: Update apt cache
         ansible.builtin.apt:
         update_cache: yes
       
   -  Outcome: The system is aware of all the latest packages and repositories.
### Install Required Software

- Purpose: Installs Docker (and its dependencies) and Nginx on the target system.
- Task: Uses the ansible.builtin.apt module to install specified packages.
- Code:
  -  ```name: Install required software (docker and nginx)
        ansible.builtin.apt:
         name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - nginx
         state: present
  
- Outcome: Docker and Nginx are installed.
### Restart Services

- Purpose: Restarts services that might require a restart after updates.
- Task: Uses ansible.builtin.systemd to restart services if updates were made.
- Code:
    - ```name: Restart services if necessary
         ansible.builtin.systemd:
          name: "{{ item }}"
          state: restarted
         loop:
          - docker
          - nginx
         when: update_result.changed

- Outcome: Docker and Nginx services are restarted if updates were applied.

### Ensure Log Directory Exists
- Purpose: Ensures that the /var/log directory exists for storing logs.
- Task: Creates the directory if it doesn’t exist, with appropriate permissions.
- Code:
  - ```name: Ensure /var/log exists
       ansible.builtin.file:
        path: /var/log
        state: directory
        mode: '0755'
    
 - Outcome: /var/log is created if missing.

### Log Update Results
- Purpose: Captures the update results and saves them to a log file for auditing.
- 
- Task: Uses ansible.builtin.copy to save the update_result variable to a file.
- Code:
   - ```name: Log the updates results
        ansible.builtin.copy:
         content: "{{ update_result | to_nice_yaml }}"
         dest: "/var/log/ansible_patch_{{ inventory_hostname }}.log"
     
- Outcome: Each host has a dedicated log file with the update results.
- 
### Execution
- Clone the repository.
- Update the inventory file with your target hosts.
  
- Run the playbook using:
   - ansible-playbook -i inventory playbook.yml
     
### Output
- Installed and updated packages.
- Docker and Nginx configured and running.
- Log files in /var/log for review and troubleshooting.
