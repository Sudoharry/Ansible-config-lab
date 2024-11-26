# Troubleshooting Guide for Automated Patch Management and Software Installation (Docker & Nginx)

This guide documents the troubleshooting steps and practices for resolving issues while running an Ansible playbook to manage Docker and Nginx installations on Ubuntu-based systems.

## Table of Contents

1. [Check Package Manager](#1-check-package-manager)
2. [Update Apt Cache](#2-update-apt-cache)
3. [Correct Docker Repository Configuration](#3-correct-docker-repository-configuration)
4. [Add Docker GPG Key](#4-add-docker-gpg-key)
5. [Add Docker Repository](#5-add-docker-repository)
6. [Install Docker and Nginx](#6-install-docker-and-nginx)
7. [Restart Services If Necessary](#7-restart-services-if-necessary)
8. [Ensure /var/log Exists](#8-ensure-/var/log-exists)
9. [Log the Update Results](#9-log-the-update-results)
10. [Directory Creation for Logs](#10-directory-creation-for-logs)
---

## 1. Check Package Manager

### **Issue**:
Unable to install Docker or Nginx due to incorrect package manager detection.

### **Action Taken**:
Verified that the system was using `apt` as the package manager and applied appropriate repository configuration.

### **Details**:
We ensured the package manager used was `apt`, which is required for adding the Docker and Nginx repositories on Ubuntu.

---

## 2. Update Apt Cache

### **Issue**:
Docker or Nginx packages not found during installation.

### **Action Taken**:
Ran the `apt update` command to refresh package sources and ensure the latest package lists were available.

### **Details**:
yaml
ansible.builtin.apt:
  update_cache: yes

## 3. Correct Docker Repository Configuration

### **Issue**:
Docker repository URL issues due to incorrect configuration in Ansible playbook.

### **Action Taken**:
We added the Docker repository using `ansible.builtin.apt_repository`. The repository URL must match the correct Ubuntu version codename (such as `focal`, `bionic`, etc.). The `signed-by` option was used to point to the GPG key that was downloaded.

### **Details**:
To avoid repository misconfigurations, we ensured the repository configuration was dynamic by using `ansible_facts.lsb.codename` to automatically pick the correct Ubuntu release codename.

yaml
ansible.builtin.apt_repository:
  repo: "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/docker.asc] https://download.docker.com/linux/ubuntu {{ ansible_facts.lsb.codename }} stable"
  state: present

## 4. Add Docker GPG Key

### **Issue**:
GPG errors related to missing or invalid Docker GPG key when attempting to install Docker or update the repository.

### **Action Taken**:
The Docker GPG key was downloaded from Docker’s official repository and placed into the trusted sources directory (`/etc/apt/trusted.gpg.d/`). We used the `ansible.builtin.get_url` module to securely fetch the key and ensure it was available for package verification.

The GPG key is necessary for ensuring the authenticity of the Docker packages that are downloaded during installation.

### **Details**:
In this step, we ensured the Docker GPG key was correctly added to the trusted list by using the following Ansible task:

yaml
ansible.builtin.get_url:
  url: https://download.docker.com/linux/ubuntu/gpg
  dest: /etc/apt/trusted.gpg.d/docker.asc

## 5. Add Docker Repository

### **Issue**:
The Docker repository was not added correctly to the system’s APT sources, preventing the installation of Docker packages. This could occur if the repository URL is malformed or not configured with the correct signature (GPG key).

### **Action Taken**:
The Docker repository was added to the system’s APT sources list using the `ansible.builtin.apt_repository` module. The repository URL was constructed dynamically based on the system's codename (such as "focal" for Ubuntu 20.04). Additionally, the GPG key was explicitly referenced using the `signed-by` option to ensure the repository is signed and secure.

### **Details**:
In this step, the Docker repository URL was dynamically configured to match the system’s Ubuntu codename (e.g., "bionic", "focal", etc.). The repository was added with the `signed-by` option that links the Docker GPG key to ensure security.

yaml
ansible.builtin.apt_repository:
  repo: "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/docker.asc] https://download.docker.com/linux/ubuntu {{ ansible_facts.lsb.codename }} stable"
  state: present

## 6. Install Docker and Nginx

### **Issue**:
The playbook failed to install Docker and Nginx, either because the package names were incorrect or the repositories were not properly configured before attempting the installation.

### **Action Taken**:
The Docker and Nginx packages were installed using the `ansible.builtin.apt` module. The installation was done after ensuring that the Docker repository was correctly added and the system package cache was updated.

### **Details**:
In this step, the installation of Docker and Nginx was triggered by using the `apt` module to install the required software. The playbook ensures that both Docker and Nginx are present on the system. The task is designed to only install the packages if they are not already installed or if they are out of date.

yaml
ansible.builtin.apt:
  name:
    - docker
    - nginx
  state: present

## 7. Restart Services If Necessary

### **Issue**:
After updating or installing packages (such as Docker and Nginx), the services might need to be restarted to apply changes, especially when new versions of software are installed or configurations are modified. The playbook should ensure that Docker and Nginx services are restarted if any updates or changes were made during the execution of the playbook.

### **Action Taken**:
The playbook checks if any updates were made to the packages (via the `update_result` variable) and restarts the Docker and Nginx services if changes were detected. This ensures that the services are running the latest version after an update.

yaml
- name: Restart services if necessary
  ansible.builtin.systemd:
    name: "{{ item }}"
    state: restarted
  loop:
    - docker
    - nginx
  when: update_result.changed

## 8. Ensure /var/log Exists

### **Issue**:
The `/var/log` directory is a crucial part of the system, where logs for various services and applications are stored. It is essential to ensure that this directory exists before proceeding with logging or monitoring tasks. In case the directory does not exist, it must be created to avoid any issues with logging or service operation.

### **Action Taken**:
The playbook checks if the `/var/log` directory exists. If it does not exist, the playbook creates the directory with appropriate permissions. This ensures that logging can occur without errors and that the system has a valid location for logs.

yaml
- name: Ensure /var/log exists
  ansible.builtin.file:
    path: /var/log
    state: directory
    mode: '0755'

## 9. Log the Update Results

### **Issue**:
When running automated patch management or system updates, it's essential to capture and log the results for troubleshooting, auditing, and monitoring purposes. The log should contain relevant information about the success or failure of the updates to ensure that any issues can be traced back easily.

### **Action Taken**:
The playbook uses the `ansible.builtin.copy` module to log the results of the update process into a log file. The result is stored in a human-readable YAML format for clarity. This provides an audit trail of the changes made during the update process, which is crucial for debugging or reviewing update histories.

yaml
- name: Log the updates results
  ansible.builtin.copy:
    content: "{{ update_result | to_nice_yaml }}"
    dest: "/var/log/update_results.log"
    mode: '0644'

## 10. Directory Creation for Logs

### **Issue**:
During system maintenance or software installations, certain directories, such as `/var/log`, may not exist or may lack the appropriate permissions. This can lead to errors when creating log files or writing logs. Ensuring that the `/var/log` directory exists with proper permissions is essential for reliable logging.

### **Action Taken**:
The playbook uses the `ansible.builtin.file` module to verify that the `/var/log` directory exists and has the correct permissions. If it doesn't exist, the task creates it with the specified permissions.

yaml
- name: Ensure /var/log exists
  ansible.builtin.file:
    path: /var/log
    state: directory
    mode: '0755'
