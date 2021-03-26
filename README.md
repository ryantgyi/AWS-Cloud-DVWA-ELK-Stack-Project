# AWS-Cloud-DVWA-ELK-Stack-Project

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Diagrams/AWS%20Cloud%20Network%20Diagram.png?raw=true)


This will be a comprehensive guide to create your AWS network, install docker and ansible, and deploy DVWA, ELK stack, filebeat, and metric beat to your various VM's inside your network. We will be using AWS CloudFormation to deploy our network infrastructure quickly. Once our network has been deployed, we can choose to edit it further if more is required from our network. 

## Understand the Topology

As show in the diagram above, our network is configured into 4 different subnets in our VPC. Our public subnets (Public1) have our instances that we can connect to over the internet, while our private subnets (Private1) contain services that we do not want people from the internet accessing. Our public subnets are accessible from the internet via our internet gateway at the front of our network. We can connect to our VMs/servers located in our private subnet by connecting to our public VM, followed by connecting to our private VM via SSH. This is facilitated by our NAT Gateway. We also have a load balancer to distribute loads to our webservers to allow us to mitigate DDoS attacks, which will allow us to connect to one webserver is the other one is offline. While we don't have any VMs/servers inside a second availability zone, we can adapt quickly to create VMs/servers there to create more redundancy within our network in case one zone goes offline. 

| Operating System 	|       Name      	|    Subnet    	| Access Policy 	| Security Group 	|   Function  	|
|:----------------:	|:---------------:	|:------------:	|:--------------:	|:--------------:	|:-----------:	|
|  Ubuntu/t2.micro  |  DVWA 1 Server  	| 10.10.2.0/24 	|     Private    	|  ELK Server-SG  |    Server   	|
|  Ubuntu/t2.micro  |  DVWA 2 Server  	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|  Ubuntu/t2.medium | ELK Stack Server	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|      Windows     	| Windows Machine 	| 10.10.0.0/24 	|     Public     	|   Windows-SG   	|   Viewing DVWA/Kibana 	  |
|    Amazon Linux   | Ansible/Jumpbox 	| 10.10.0.0/24 	|     Public     	|   Jumpbox-SG   	|   Gateway/Deployment   	|

## Purpose of the network

The purpose of our network is to deploy two DVWA machines with filebeat and an ELK server with metricbeat. Using the files listed in the Ansible folder, we will be using ansible and docker to quickly deploy these services to our VMs in our private subnet from our public jumpbox. The purpose of this deployment method is to be able to quickly deploy these services to many machines at once, eliminating the need to set each machine up individually, ultimately saving time and money. 

## Access policies

  -  The machines on our public subnet (Public1) are available to be access via the internet 
  -  You will need the private key that helped create the VM to be available on the machine you are trying to connect from
  -  To connect to our the private VMs/servers, you will need to connect to them after connecting to a public VM

## DVWA/ELK Configuration

Ansible is being used to automate the configuration of our DVWA/ELK servers. Aside from minor network and file edits, no manual installation or configuration is needed. Our playbook files will be downloading, installing, and configuring our DVWA/ELK servers on our target machines. As seen in the diagram above, these machines are located on our private subnet. It is important that our target machines for our DWVA/ELK servers have an ubuntu operating system, otherwise the code inside our playbook files will not run. Our ELK server is set up to monitor the following machines
  -  DWVA1 with filebeat - 10.10.2.x/24
  -  DVWA2 with filebeat - 10.10.2.x/24
  -  Note: The initial deployment of our DVWA machines do not contain filebeat. This will be installed after our deployment of our DVWA machines.

## Using our Ansible playbooks
In order to use our playbook files effectively, we need to have docker installed on a machine, in this case our Jumpbox VM. Once you have docker installed on your VM:
  -  Copy the relevant .yml files and private key(s) onto your jumpbox
  -  Deploy your ansible container
  -  Copy your .yml files to your ansible container along with the private key(s) into your ansible container
  -  Edit the /etc/ansible/hosts with the relevant IPs and headers
  -  Edit the /etc/ansible/ansible.cfg with the correct remote_user
  -  Deploy the relevant playbook files (ansible_config.yml -> install-elk.yml -> filebeat-playbook.yml -> metricbeat-playbook.yml)

# Step by step process

## Step 1: Using CloudFormation to create out VPC Network
  -  Go to https://aws.amazon.com/cloudformation/ click "Create Stack"
  -  Select "Upload a template"
  -  Upload the file in the repository "Basic_Network_Cloud_Formation.yaml" and select next
  -  Name the network stack and select next until you can launch your instance

Before continuing on to make our VMs, we need add some functionality to our network.

  -  Search for VPC in the top search bar
  -  On the left side bar, search for subnet
  -  Select the subnets "Public1" and "Public2"
  -  Near the top/top right, select the "Actions" and select the option "Modify auto-assign IP settings"
  -  Select the checkbox to enable auto-assign public IPv4 address

## Step 2: Setting up our Amazon VM's
It is important on this step to create 3 ubuntu VMs inside our subnet "Private2". The jumpbox will be our amazon linux, located inside our subnet "Public1". First we will set up our Amazon linux.

  -  Using the search bar at the top, navigate to EC2. Using the left side bar, click on "Instances".
  -  In the top right, click on "Launch instances".
  -  On Step 1: Select "Amazon Linux 2 AMI (HVM), SSD Volume type.
  -  On Step 2: Select t2.micro and click next.
  -  On Step 3: 
      -  Under the "Network" drop down, select VPC1.
      -  Under the "Subnet", select "Public1" and click next.
  -  Keep clicking next until you hit "Step 6: Configure Security Group".
  -  On Step 6: Click "Create a new security group".
      -  Change "Security Group name" from launch-wizard to "Jumpbox - SG".
      -  Click "Add Rule" under SSH and select HTTP. Once HTTP is added to the list, click the dropdown under "Source" and select "Anywhere".
      -  Click "Review and Launch" and click "Launch".
      -  Note: SSH and HTTP should be enabled
  -  If you do not have an existing key pair, click "Create a new key pair". Put in a name for your pair name and click "Download Key Pair".
  -  Click "Launch Instance".
      - Note: We will use this key pair in the rest of the guide, but you can create a new key if you want to make each VM accessible with it's own key pair. However, if you lose your any key file, you will be unable to access the VM again and will need to delete it and recreate it again.
  - Now that your VM has been created, navigate back to the left side bar and click "Instances". Rename your new VM to "Jumpbox" to keep track of it.

Now that we have our Jumpbox/Amazon Linux machine, we will create the Windows VM.

  -  Click launch instance.
  -  On Step 1: Choose an Amazon Machine Image (AMI), scroll down and select "Microsoft Windows Server 2019 Base" and click next.
  -  On Step 2: Select t2.micro and click next.
  -  On Step 3: 
      -  Under the "Network" dropdown, select VPC1 
      -  Under the subnet dropdown, select "Public1" and click next.
  -  Click next until you get to Step 6: Configure Security Group.
  -  On Step 6: click "Create a new security group".
      -  Change "Security Group name" from launch-wizard to "Windows - SG".
      -  Click "Add Rule". Under the "Type" dropdown, change "Custom TCP Rule" to HTTP. Under the "Source" dropdown, select "Anywhere" and click next.
      -  Note: RDP and HTTP should be enabled
  -  Click "Review and Launch" and click "Launch".
  - For "Choose an existing key pair", select the key that you made or used during your Amazon Linux VM creation.
  - Now that your VM has been created, navigate back to the left side bar and click "Instances". Rename your new VM to "Windows" to keep track of it.

Now that our public VMs have been made, we will start making our Private VMs.

  -  Click launch instance.
  -  On Step 1: Choose an Amazon Machine Image (AMI), scroll down and select the latest Ubuntu server "Ubuntu Server..." and click next.
  -  On Step 2: Select t2.micro and click next.
  -  On Step 3: 
      -  Under "Number of instances", change 1 to 2. (Note: This step is not included in the other VM launch steps)
      -  Under the "Network" dropdown, select VPC1. 
      -  Under the subnet dropdown, select "Private1" and click next.
  -  Click next until you get to Step 6: Configure Security Group.
  -  On Step 6: click "Create a new security group".
      -  Change "Security Group name" from launch-wizard to "Webserver - SG".
      -  Click "Add Rule". Under the "Type" dropdown, change "Custom TCP Rule" to HTTP. Under the "Source" dropdown, select "Anywhere" and click next.
      -  Note: SSH and HTTP should be enabled
-  Click "Review and Launch" and click "Launch".
  - For "Choose an existing key pair", select the key that you made or used during your Amazon Linux VM creation.
  - Now that your VMs has been created, navigate back to the left side bar and click "Instances". Rename your new VMs to "DVWA1" and "DVWA2" to keep track of them.

Now that we have made our DVWA machines, we will make our ELK machine. This one has one important difference, the amount of memory it has will be t2.medium instead of t2.micro.

  -  Click launch instance.
  -  On Step 1: Choose an Amazon Machine Image (AMI), scroll down and select the latest Ubuntu server "Ubuntu Server..." and click next.
  -  On Step 2: Select t2.medium and click next.
  -  On Step 3: 
      -  Under the "Network" dropdown, select VPC1. 
      -  Under the subnet dropdown, select "Private1" and click next.
  -  Click next until you get to Step 6: Configure Security Group.
  -  On Step 6: click "Create a new security group".
      -  Change "Security Group name" from launch-wizard to "Elk Stack - SG".
      -  Click "Add Rule". Under the "Type" dropdown, change "Custom TCP Rule" to HTTP. Under the "Source" dropdown, select "Anywhere".
      -  Click "Add Rule". Under "Port range", type 5044. Under the "Source" dropdown, select "Anywhere".
      -  Click "Add Rule". Under "Port range", type 5601. Under the "Source" dropdown, select "Anywhere".
      -  Click "Add Rule". Under "Port range", type 9200. Under the "Source" dropdown, select "Anywhere".
      -  Click "Add Rule". Under "Port range", type 9300. Under the "Source" dropdown, select "Anywhere".
      -  Click "Add Rule". Under "Port range", type 9400. Under the "Source" dropdown, select "Anywhere".
      -  Note: Ports 22, 80, 5044, 5601, 9200, 9300, 9600 should be enabled.
  -  Click "Review and Launch" and click "Launch".
  - For "Choose an existing key pair", select the key that you made or used during your Amazon Linux VM creation.
  - Now that your VM has been created, navigate back to the left side bar and click "Instances". Rename your new VM to "ELK Stack" to keep track of it.

## Step 3: Create the Load Balancer

  -  Navigate back to the EC2 page. On the left side bar, scroll down and select Load Balancers.
  -  Click on "Create Load Balancer".
  -  Select "Application Load Balancer" and click "Create".
  -  Step 1:
      -  Under "Name", put a meaningful name, or you can name it "Load Balancer 1"
      -  Under "Scheme", select "internal"
      -  For Availability Zone under VPC, under the dropdown, select VPC1.
          -  Under availability zones, select both and choose "Private1" and "Private2".
      - Select next
  - Step 2: Select next
  - Step 3: Configure Security Groups
      -  Select "Create a new security group".
      -  Under "Type" for the first rule, change it to HTTP and click next.
  -  Step 4: Configure Routing
      -  Under "Target Group", select "New target group"
      -  Name the target group. If you can't think of anything, put "DVWA target group" or "TG1" and click next.
      -  Under the "Name", select DVWA1 and DVWA2 and click "Add to registered" and click next.
  -  Click next until create.

Now that we have set up our load balancer, we are ready to connect and configure our instances. 

## Step 4: Configuring your Ansible playbook files

Now that we have our network up and running, we will need to configure out filebeat.yml and metricbeat.yml files to point to the correct IP address of our VMs. Navigate back to your Amazon EC2 instance page to have access to your VM IP addresses. You will need the private IP addresses of your DVWA1, DVWA2, and ELK Stack VMs.

  -  Open the filebeat.yml file using notepad or another code editor. Ctrl+F and search for output.elasticsearch and search for the code listed below

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Images/filebeat%20elasticsearch%20ip%20edit.PNG?raw=true)

  -  Changed the highlighted IP to your DVWA1 server IP, make sure to keep the port.
  -  Ctrl+F and search for setup.kibana and search for the following code

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Images/filebeat%20kibana%20ip%20edit.PNG?raw=true)

  -  Change the highlighted IP to your DVWA1 server IP, make sure to keep the port
  -  Save and exit

Now that we have configured the filebeat.yml file, we will configure the metricbeat.yml file.

  -  open the metricbeat.yml file using notepad or another code editor. Scroll down and look for the following code

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Images/metricbeat%20kibana%20ip%20edit.PNG?raw=true)

  -  Change the highlighted IP to your ELK server IP. Make sure to keep the port
  -  Scroll down and search for the following code

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Images/metricbeat%20elasticsearch%20ip%20edit.PNG?raw=true)

  -  Change the highlighted IP to your ELK server IP. Make sure the keep the port
  -  Save and exit

## Step 5: Connecting to our Virtual Machines for Deployment

  -  First we need to open a Bash/command prompt and navigate to where the private key we downloaded on our first VM creation.
      -  If you haven't moved key, it should be located inside your downloads folder (use command - cd Downloads)
      -  If you haven't yet, download the files in the relevant ansible folders for what you want to deploy
  -  Navigate back to our EC2 instances page. Select our Jumpbox, and click the orange "Connect", click SSH client. Copy the SSH command by either highlighting and copying or by clicking the small double box icon at the beginning of the command
      -  The command should look like "ssh -i <key name.pem> <user>@<destination>"
  -  Before connecting, we want to transfer the private key and the ansible files onto our jumpbox
      -  Use the command scp -i <key name.pem> <file(s) to be transferred separated by a space> <user>@<destination>:<path/to/home/directory>
      -  example: scp -i "AWS-CloudKey.pem" AWS-CloudKey.pem ansible_config.yml ec2-user@ec2-3-139-108-207.us-east-2.compute.amazonaws.com:/home/ec2-user
          -  In this example, we transferred AWS-Cloudkey.pem and ansible_config.yml in one command
          -  For the Amazon Linux, the home directory will always be /home/ec2-user
          -  The <user>@<destination> will look "ec2-user@ec2-3-139-108-207.us-east-2.compute.amazonaws.com" and be the exact same in the ssh command you just copied. 
  -  Now that we have transfered the files, log into your jumpbox machine using the ssh command that you copied.

Now that we are logged into our machines, we will run the following commands.
  -  sudo yum update
  -  sudo yum upgrade

Now that we have upgraded our jumpbox, go back to the Amazon instances page and connect to each one of your ubuntu machines using the orange "connect" button and copy/pasting the ssh commands into the terminal. Note: You must be logged into your jumpbox machine to access the ubuntu machines as they are located on your private subnet. Once you have logged onto your first ubuntu machine, run the following commands.
  -  sudo apt-get update
  -  sudo apt-get upgrade
  -  exit

Once you hit enter on the exit command, this will put you back into Jumpbox VM. Now log into the other two ubuntu VMs and run the same two commands above. Upon updating and upgrading your third ubuntu machine and exiting back to your jumpbox, we will now start to install docker.
  -  sudo yum install docker -y

Now that docker has been installed, we need to make a daemon.json file to make sure our docker containers will end up on our subnet IP addresses. 
  -  sudo nano /etc/docker/daemon.json
      -  Once you have created the daemon.json file, copy the content in the daemon.json file in the Ansible folder in the Git Repository and paste it into the .json file you have just opened. 
      -  Save and exit

Now that we have created our daemon.json file, we can start or restart docker.
  -  sudo service docker start
  -  sudo service docker restart

You can check to see if the service is running using
  -  sudo service docker status

How that docker is up and running, we are going to pull a docker image and use that image to deploy a container.
  -  sudo docker pull cyberxsecurity/ansible
  -  sudo docker run -ti cyberxsecurity/ansible bash

This will move us from our jumpbox into the ansible container itself. It is important you do not exit out of this container or everything that you move into this container will be deleted and you have to restart using the run command above. Since we cannot exit out of this container without deleting it, we will need to open up a second bash/command prompt terminal in order to move files from out jumpbox VM into our ansible container. After navigating back to your downloads folder (or where your private keyu is), log in to your jumpbox VM using the ssh command. We will now copy the files you moved into your jumpbox using the scp command into your docker container.
  -  sudo docker ps

This will show you an output of all docker containers running. Since we only have one container running, look for the container id in the "sudo docker ps" output. If for some reason you have more than one container from exiting earlier and aren't sure which one is which, look at your original bash terminal that is logged into your container and copy the container id after root@<container id>. Now that you have the container id, we will copy each file using one command. Unlike the scp command, you cannot copy more than one file at time so you must run this command more than once.
  -  sudo docker cp <file name> <container id>:/root
  -  example: sudo docker cp AWS-CloudKey.pem fb9cjkdj28:/root

You must copy all the files from their respective ansible folders in order to run each service.

Now that our files have been transferred, we can start editing the container files to deploy our services to their respective VMs. Moving back to our ansible container bash terminal (the first terminal), run the following command.

  -  nano /etc/ansible/hosts

Now that we are inside the file, we need to set the proper headers and IPs.

  -  Scroll down and search for "## [webservers]. Remove the comments (##) and the space before the [webservers]. This makes [webservers] our header.
  -  On two new lines right under [webservers], put the IP addresses of your DVWA1 and DVWA2 VMs
  -  On a new line right under your DVWA2 IP address, put a new header just like [webservers] called "[elkservers]" (without quotes). This makes [elkservers] another new header.
  -  One a new line right under [elkservers], put the IP address of your ELK server VM

Now that we have configured out /etc/ansible/hosts file, we need to configure the other file. Use the following command to access the ansible.cfg file

  -  nano /etc/ansible/ansible.cfg
      -  Hit Ctrl+W and type "remote_user"
      -  remove the comment (#) before remote_user and change root to "ubuntu"
      -  the line should read
          -  remote_user=ubuntu
  -  Now that our ansible files have been configured, we need to ssh back into our VMs again. Using the Amazon connect/ssh command, log into your DVWA1, DVWA2, and ELK servers from your ansible container. When you log in, just exit back out back into the ansible container again before using ssh to log into another VM. While this step might be optional, it prevented me from running into an error.
Now that we have logged into our VM machines and back into our ansible container, we can deploy our ansible playbooks. Run the following command
  -  ansible-playbook ansible_config.yml --key-file <key name.pem>
  -  example: ansible-playbook ansible_config.yml --key-file AWS-CloudKey.pem

This will start deploying our DVWA services onto our DVWA machines. After that is finished deploying, we can deploy our ELK playbook.
  -  ansible-playbook install-elk.yml --key-file <key name.pem>
  -  example: ansible-playbook install-elk.yml --key-file AWS-CloudKey.pem

Once ELK has been deployed to our ELK Server VM, we can deploy filebeat. Run the following command.

  -  ansible-playbook filebeat-playbook.yml --key-file <key name.pem>
  -  example: ansible-playbook filebeat-playbook.yml --key-file AWS-CloudKey.pem

Once filebeat has been deployed, we can deploy metricbeat. Run the following command.
  -  ansible-playbook metricbeat-playbook.yml --key-file <key name.pem>
  -  example: ansible-playbook metricbeat-playbook.yml --key-file AWS-CloudKey.pem

NOTE: If filebeat or metricbeat do not deploy, do not exit out or you will have to reconfigure your ansible container again starting at the "sudo docker run -ti cyberxsecurity/ansible" command. Remove the playbook files using the following commands for whichever service did not deploy successfully. You will have to edit the playbook file(s) on your local computer and copy them to your Jumpbox VM and from there to your ansible container again.
  -  rm filebeat-playbook.yml
  -  rm metricbeat-playbook.yml

NOTE: Do not close your ansible container bash until you have successful deployed both filebeat and metricbeat

## Step 6: Logging into your DVWA and ELK servers on Windows

Now that your DVWA and ELK servers have been deployed, you can log into them on your Windows VM. Go back to your Amazon EC2 instance page and click connect to your Windows VM.   
  -  Click on "RDP client" and click "Download remote desktop file".
  -   Look lower and click on "Get password". 
  -   Click on "Browse" and select your private key .pem file that you used to create your Windows VM. 
  -   Once you select your file, click "Decrypt Password" and copy the decrypted password by highlighting and copying or clicking on the two square icon to the left of the password. 
  -   Open your downloads folder and open the file titled "Windows" with the type "Remote Desktop Connection". 
  -   Paste your copied password into the prompt and connect to your windows machine. 

Now that we are in Windows, we need to disable some security settings to download Chrome and effectively see our services (they do not run on IE)
  -   Once your desktop loads, open "Server Manager"
  -   Click on "Local server". 
  -   Find the setting titled "IE Enhanced Security Configuration"
  -   Turn both setting from on to "off" 
  -   Open your browser and download chrome

Now that we have Chrome, we can connect to our DVWA machines. Minimize your RDP and go back to your Amazon EC2 page. Go to the left side bar and go to your load balancer that you created. Click on the load balancer and located the "DNS name" under description. Copy the DNS name (Should start with "internal"). Paste the DNS name you copied into Chrome on your Windows VM. If the DVWA webpage loads, everything is good! If not, try the private IP of your DVWA1 machine or DVWA2 machine.

Now that you have connected to your DVWA servers, we will connect to our elk server. On a new tab in Chrome on your Windows machine, type the private IP of your elk VM in the search bar followed by :5601.

  -  <ELK Stack private IP>:5601
  -  example: 10.10.2.X:5601

If the Kibana webpage loads, everything is good!

## Troubleshooting Filebeat and Metricbeat deployment

If filebeat or metricbeat do not deploy we need to reconfigure the filebeat-playbook.yml file and/or the metricbeat-filebeat.yml file.

  -  On our Windows VM, open chrome and reconnect to Kibana/Elk server using 10.10.2.X:5601
  -  Click "Add log data"
  -  Click "Logstash data"
  -  Right under "Getting started", click on "DEB" to open the linux commands needed
  -  On your local machine, open the filebeat-playbook.yml file and follow the instructions under "Getting Started"

When your done, go back to the kibana home page.
  -  Click on "Add metrics"
  -  Click on "Kibana metrics"
  -  Right under "Getting started", click on "DEB" to open the linux commands needed
  -  On your local machine, open the metricbeat-playbeat.yml file and follow the instructions under "Getting Started"

Now that the playbook files have been configured, open up a second bash script. Log into your Jumpbox VM using the Amazon connect ssh command.
  -  rm filebeat-playbook.yml
  -  rm metricbeat-playbook.yml
  -  exit

Copy the new filebeat-playbook.yml file and the metricbeat-playbook.yml file into your Jumpbox VM using the scp command from earlier and log into your Jumpbox VM using the Amazon connect ssh command. Now that the updated files are on your Jumpbox, you can copy them using the following commands

  -  sudo docker cp filebeat-playbook.yml <docker container id>:/root
  -  sudo docker cp metricbeat-playbook.yml <docker container id>:/root

Now moving back to your ansible container bash terminal, run the following commands again.

  -  ansible-playbook filebeat-playbook.yml --key-file <key file.pem>
  -  ansible-playbook metricbeat-playbook.yml --key-file <key file.pem>

This should deploy the services properly.
