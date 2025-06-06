# Deploy an Ubuntu Web Server in Azure

1. In the Azure Portal, click Create a resource. Choose Virtual Machine and click Create.

1. Review the avilable options, and configure the following:

|Option|Value|
|---|---|
|Subscription|Free Tier|
|Resource Group|Existing or create|
|Virtual machine name|Your choice|
|Region|Choose closest|
|Availability Options|No infrastructure redundancy required|
|Security Type|Trusted launch VMs|
|Image|Ubuntu Server 24.04 LTS - x64 Gen2|
|Architecture|x64|
|Run with Azure Spot discount|No|
|Size|Standard_B1\|s|
|Authentication type|SSH public key|
|Username|Your choice|
|SSH public key source|Generate new key pair|
|SSH key type|RSA SSH Format|
|Key pair name|Your choice|
|Public inbound ports|Allow selected ports|
|Select inbound ports|SSH 22 & HTTP 80|

3. Click `Next: Disks`

Review the avilable options, and configure the following:

|Option|Value|
|---|---|
|OS disk size|image default|
|OS disk type|Standard HDD (LRS)|
|Delete with VM|Yes|
|Key management|Platform managed|

4. Click `Next: Networking`

5. Review the available options, but leave everything as default.

6. Click `Next: Management`

7. Review the available options, but leave everything as default.

8. Click `Next: Monitoring`

9. Review the available options, but leave everything as default.

10. Click `Next: Advanced`

11. Review the available options, but leave everything as default.

        Note the 'Custom data' and 'User data' fields, these allow you to provide a configuration script which is passed through to the VM while it is being provisioned.
        
        You can define configuration options, deploy software, map storage, and many other tasks.
        
        These will all be completed before the machine is marked as ready.

12. Click `Next: Tags`

        Tags are key value pairs that are used to attach meta-data to your resources in the form of key-value pairs. They can be used to assist in resource management, cost management, monitoring, running reports, and many other tasks. 

13. Click `Next: Review + create`

Confirm your selected configuration options, then click 'create'.

14. When prompted click `Donwload key and deploy resources`; Monitor the various resources being created - notice that the VM resource has a number of additional resources upon which it is dependent.

15. Once deployment has finished click `Go to resouce`.

16. Due to Windows being finnicky about permissions, it is recommended that you connect to your VM using Bash. You can either use your Linux CentOS VM, or open a Git Bash terminal in VSCode. Open your Terminal in, or navigate to the location where your prive key is stored. Connect to your VM with `ssh -i [keyname] [username]@[public IP]`.

17. Once connected to your VM install NGINX with `sudo apt install nginx -y`

18. Confirm installation with `sudo systemctl status nginx`, then view the test page in your browser by going to `http://[public IP]`.

19. Create a custom `index.html` file at `/var/www/html`.

20. Once you have finished don't forget to delete your web server and associated resources (including Resource Groups).

