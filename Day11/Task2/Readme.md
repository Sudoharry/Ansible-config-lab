## This is Automated user management

### Task: Created a playbook to manage users and groups on your servers.

### Implemented features:

    - Add a group (devops)
    - Add multiple (users1,user2) to the group
    - Set user passwords (hashed for security)

### Create a file for password variable vars.yaml
    - Add your passwords for all the users in the devops group
    - Pass the variable in the playbook using vars_files:  -vars.yml

## Also, add a debug variable to verify is it accessible or not, add a debug task to your playbook

    - name: Debug variables
      debug:
        msg:
         - "Password1: {{ password1 }}"
         - "Password2: {{ password2 }}"
## Run the playbook to ensure the variables are correctly set
  
    ansible-playbook -i inventory.ini user-management.yaml

## To encrypt the password files which is stored in the variable

   - ansible-vault encrypt vars.yml
   - Set vault password for access

## Run the playbook using this command now
   
     ansible-playbook -i inventory.ini user-management.yaml --ask-vault-pass 


## Check whether the group along with users created or not

  - Use Ad-hoc command:

      ansible all -i inventory.ini -m shell -a "getent group devops"
       
       ubuntu@65.0.61.81 | CHANGED | rc=0 >>
       devops:x:1001:
       ubuntu@13.201.21.94 | CHANGED | rc=0 >>

       devops:x:1001: 
## Use Encrypted Variables with Ansible Vault, 
   - For better security, store passwords in encrypted files using Ansible Vault.
 
1) Encrypt a file
  
  - ansible-vault encrypt vars.yml
	
2) Include Encrypted File in Playbook:

  vars_files:
    - vars.yml

3) Run the Playbook

  ansible-playbook -i inventory.yml playbook.yml --ask-vault-pass

## Use adhoc-commands:

  - After running the playbook, you can use Ansible ad-hoc commands to verify the users and groups created.

      ansible all -i inventory.yml -m shell -a "getent group devops"

   - Check if the users user1 and user2 were created:
     ansible all -i inventory.yml -m shell -a "getent passwd user1"
     ansible all -i inventory.yml -m shell -a "getent passwd user2"

## Check Users and Groups Manually on the Hosts

    - You can also SSH into the hosts and check manually:

      Check if the devops group exists:

      grep devops /etc/group
      
   Check if the users user1 and user2 exist:
       id user1
       id user2
      If the users exist, their details (UID, GID, etc.) will be displayed.


