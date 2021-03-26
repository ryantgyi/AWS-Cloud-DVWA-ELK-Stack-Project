# AWS-Cloud-DVWA-ELK-Stack-Project

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Diagrams/AWS%20Cloud%20Network%20Diagram.png?raw=true)


This will be a comprhensive guide to create your AWS network, install docker and ansible, and deploy DVWA, ELK stack, filebeat, and metric beat to your various VM's inside your network. We will be using AWS CloudFormation to deploy our network infrastructure quickly. Once our network has been deployed, we can choose to edit it further if more is required from our network. 

## Understand the Topology

As show in the diagram above, our network is configured into 4 different subnets in our VPC. Our public subnets (Public1) have our instances that we can connect to over the internet, while our private subnets (Private1) contain services that we do not want people from the internet accessing. Our public subnets are accessible from the internet via our internet gateway at the front of our network. We can connect to our VMs/servers located in our private subnet by connecting to our public VM, followed by connecting to our private VM via SSH. This is facilitated by our NAT Gateway. We also have a load balancer to distribute loads to our webservers to allow us to mitigate DDoS attacks, which will allow us to connect to one webserver is the other one is offline. While we don't have any VMs/servers inside a second availability zone, we can adapt quickly to create VMs/servers there to create more redundancy within our network in case one zone goes offline. 

| Operating System 	|       Name      	|    Subnet    	| Access Policy 	| Security Group 	|   Function  	|
|:----------------:	|:---------------:	|:------------:	|:--------------:	|:--------------:	|:-----------:	|
|  Ubuntu/t2.micro  |  DVWA 1 Server  	| 10.10.2.0/24 	|     Private    	|  ELK Server-SG  |    Server   	|
|  Ubuntu/t2.micro  |  DVWA 2 Server  	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|  Ubuntu/t2.medium | ELK Stack Server	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|      Windows     	| Windows Machine 	| 10.10.0.0/24 	|     Public     	|   Windows-SG   	|   Viewing DVWA/Kibana 	  |
|    Amazon Linux   | Ansible/Jumpbox 	| 10.10.0.0/24 	|     Public     	|   Jumpbox-SG   	|   Gateway/Deployment   	|

## Purose of the network

The purpose of our network is to deploy two DVWA machines with filebeat and an ELK server with metricbeat. Using the files listed in the Ansible folder, we will be using ansible and docker to quickly deploy these servies to our VMs in our private subnet from our public jumpbox. The purpose of this deployment method is to be able to quickly deploy these servies to many machines at once, eliminating the need to set each machine up individually, ultimately saving time and money. 

## Access policies

  -  The machines on our public subnet (Public1) are available to be access via the internet 
  -  You will need the private key that helped create the VM to be available on the machine you are trying to connect from
  -  To connect to our the private VMs/servers, you will need to connect to them after connecting to a public VM

## DVWA/ELK Configuration

Ansible is being used to automate the configuration of our DVWA/ELK servers. Aside from minor network and file edits, no manual installation or configuration is needed. Our playbook files will be downloading, installing, and configuring our DVWA/ELK servers on our target machines. As seen in the diagram above, these machines are located on our private subnet. It is important that our target machines for our DWVA/ELK servers have an ubuntu operating system, otherwise the code inside our playbook files will not run. Our ELK server is set up to monitor the following machines
  -  DWVA1 with filebeat - 10.10.2.x/24
  -  DVWA2 with filebeat - 10.10.2.x/24
  -  Note: The inital deployment of our DVWA machines do not contain filebeat. This will be installed after our deployment of our DVWA machines.

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
  -  Select the checkbox to enable auto-assign public public IPv4 address

## Step 2: Setting up our Amazon VM's
It is imporant on this step to create 3 ubuntu VMs inside our subnet "Private2". The jumpbox will be our amazon linux, located inside our subnet "Public1". First we will set up our Amazon linux.

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

























------------------------------------------------------------------------------------------------------------------------
Now that your network has been setup, we will start installing docker/ansible to start deploying our server applications.
  -  Let's ssh into your Jumpbox machine
  -  Run "sudo yum update" and "sudo yum upgrade" to make sure our machine is up to date
