## Deploy a Static Website
  
    Task: Write a playbook to deploy a static website on Nginx.
     Features to Implement:

   - Copy HTML/CSS files from your Ansible control node to the server.
   - Ensure the correct permissions are set.
   - Restart the Nginx service after deployment.

1) Create an inventory.ini files and add the webservers listed machines that you need to deploy the static website

2) Create a playbook using the deploy-static-website.yml

    Task need to performed:
    2.1)Install Nginx if not installed
    2.2) Create a directoty for webiste content
    2.3) Copy HTML files to the servers
    2.4) Copy Css files to the servers
    2.5) Restart Nginx servers
    2.6) Create a CSS directory  --  Add before copy html and css, as there was not directory created at the destination hosts.

3) Run the playbook
 
      ansible-playbook -i inventory.ini deploy-static-website.yml
 
4) Check whether the static website is delpoyed at targeted node

    ansible webservers -i inventory.ini -m ping 
