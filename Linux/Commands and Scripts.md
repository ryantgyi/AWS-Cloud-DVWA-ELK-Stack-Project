List of Linux commands
  - ssh -i "<AWS Key.file>" <AWS Amazon link>
      example: ssh -i "AWS-CloudKey.pem" ec2-user@ec2-18-191-212-225.us-east-2.compute.amazonaws.com
  - scp -i "<AWS Key.file>" <File(s) to transfer> :<Directory path to copy>
      - example: scp -i "AWS-CloudKey.pem" AWS-Cloudkey.pem ec2-user@ec2-18-191-212-225.us-east-2.compute.amazonaws.com:/home/ec2-user
      - example: scp -i "AWS-CloudKey.pem" ansible_config.yml ec2-user@ec2-18-191-212-225.us-east-2.compute.amazonaws.com:/home/ec2-user
      - example (combining both):  scp -i "AWS-CloudKey.pem" AWS-Cloudkey.pem ansible_config.yml ec2-user@ec2-18-191-212-225.us-east-2.compute.amazonaws.com:/home/ec2-user
  - sudo chmod 400 AWS-CloudKey.pem
  - sudo yum install docker
  - sudo service docker start
  - nano daemon.json (navigate to /etc/docker)
  - sudo service docker restart
  - sudo docker pull cyberxsecurity/ansible
  - sudo docker run -ti cyberxsecurity/ansible bash
  - (Open up second bash terminal and log in to your ec2-user VM: (2) will indicate second bash terminal)
  - (2) sudo docker ps
  - (2) sudo docker cp <file to transfer> <docker container id>:/root
      - The container id is found in the output of the "sudo docker ps" command
      - example: sudo docker cp AWS-Cloudkey.pem <docker container id>:/root
      - example: sudo docker cp ansible_config.yml <docker container id>:/root
          - You need to copy both files with two different commands
  -  nano /etc/ansible/hosts
      -  scroll down to find "## [webservers]" and remove the comments
      -  add a new line under [webservers] and add the IP(s) of your DVWA machines, save and exit
  -  nano /etc/ansible/ansible.cfg
      -  Ctrl+W and type in remote_user
      -  remove the comment before remote_user and change the user from root to ubuntu, save and exit
  - ssh into your DWVA machine(s)
      -  sudo apt-get update
      -  sudo apt-get upgrade
      -  exit
      -  Make sure not to exit out of your docker container
  - ansible-playbook <ansible playbook file> --key-file <private key.file>
      -  example: ansible-playbook ansible_config.yml --key-file AWS-Cloudkey.pem


NOTES:
  - for DWVA, ELK, filebeat, or metricbeat, you need all files inside their respective folders to be inside the ansible container before running the ansible-playbook command.


   
