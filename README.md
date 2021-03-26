# AWS-Cloud-DVWA-ELK-Stack-Project

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Diagrams/AWS%20Cloud%20Network%20Diagram.png?raw=true)


This will be a comprhensive guide to create your AWS network, install docker and ansible, and deploy DVWA, ELK stack, filebeat, and metric beat to your various VM's inside your network. We will be using AWS CloudFormation to deploy our network infrastructure quickly. Once our network has been deployed, we can choose to edit it further if more is required from our network. 

## Understand the Topology

As show in the diagram above, our network is configured into 4 different subnets in our VPC. Our public subnets (Public1) have our instances that we can connect to over the internet, while our private subnets (Private1) contain services that we do not want people from the internet accessing. Our public subnets are accessible from the internet via our internet gateway at the front of our network. We can connect to our VMs/servers located in our private subnet by connecting to our public VM, followed by connecting to our private VM via SSH. This is facilitated by our NAT Gateway. We also have a load balancer to distribute loads to our webservers to allow us to mitigate DDoS attacks, which will allow us to connect to one webserver is the other one is offline. While we don't have any VMs/servers inside a second availability zone, we can adapt quickly to create VMs/servers there to create more redundancy within our network in case one zone goes offline. 

| Operating System 	|       Name      	|    Subnet    	| Access Policy 	| Security Group 	|   Function  	|
|:----------------:	|:---------------:	|:------------:	|:--------------:	|:--------------:	|:-----------:	|
|      Ubuntu      	|  DVWA 1 Server  	| 10.10.2.0/24 	|     Private    	|  ELK Server-SG  |    Server   	|
|      Ubuntu      	|  DVWA 2 Server  	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|      Ubuntu      	| ELK Stack Server	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|      Windows     	| Windows Machine 	| 10.10.0.0/24 	|     Public     	|   Windows-SG   	|   Viewing DVWA/Kibana 	  |
|   Amazon Linux   	| Ansible/Jumpbox 	| 10.10.0.0/24 	|     Public     	|   Jumpbox-SG   	|   Gateway/Deployment   	|

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







------------------------------------------------------------------------------------------------------------------------
Now that your network has been setup, we will start installing docker/ansible to start deploying our server applications.
  -  Let's ssh into your Jumpbox machine
  -  Run "sudo yum update" and "sudo yum upgrade" to make sure our machine is up to date
  -  
