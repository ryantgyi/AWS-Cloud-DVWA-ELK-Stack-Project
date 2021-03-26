# AWS-Cloud-DVWA-ELK-Stack-Project

![Alt text](https://github.com/ryantgyi/AWS-Cloud-DVWA-ELK-Stack-Project/blob/main/Diagrams/AWS%20Cloud%20Network%20Diagram.png?raw=true)


This will be a comprhensive guide to create your AWS network, install docker and ansible, and deploy DVWA, ELK stack, filebeat, and metric beat to your various VM's inside your network. We will be using AWS CloudFormation to deploy our network infrastructure quickly. Once our network has been deployed, we can choose to edit it further if more is required from our network. 

## Understand the Topology

As show in the diagram above, our network is configured into 4 different subnets in our VPC. Our public subnets (Public1) have our instances that we can connect to over the internet, while our private subnets (Private1) contain services that we do not want people from the internet accessing. Our public subnets are accessible from the internet via our internet gateway at the front of our network. We can connect to our VMs/servers located in our private subnet by connecting to our public VM, followed by connecting to our private VM via SSH. This is facilitared by our NAT Gateway. We also have a load balancer to distribute loads to our webservers to allow us to mitigate DDoS attacks, which will allow us to connect to one webserver is the other one is offline.

| Operating System 	|       Name      	|    Subnet    	| Access Policy 	| Security Group 	|   Function  	|
|:----------------:	|:---------------:	|:------------:	|:--------------:	|:--------------:	|:-----------:	|
|      Ubuntu      	|  DVWA 1 Server  	| 10.10.2.0/24 	|     Private    	|  ELK Server-SG  	|    Server   	|
|      Ubuntu      	|  DVWA 2 Server  	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|      Ubuntu      	| ELK Stack Server	| 10.10.2.0/24 	|     Private    	|  Webserver-SG  	|    Server   	|
|      Windows     	| Windows Machine 	| 10.10.0.0/24 	|     Public     	|   Windows-SG   	|   Gateway 	  |
|   Amazon Linux   	| Ansible/Jumpbox 	| 10.10.0.0/24 	|     Public     	|   Jumpbox-SG   	|   Gateway   	|



Now that your network has been setup, we will start installing docker/ansible to start deploying our server applications.
  -  Let's ssh into your Jumpbox machine
  -  Run "sudo yum update" and "sudo yum upgrade" to make sure our machine is up to date
  -  
