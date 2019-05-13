# Ansible_deployment
System Deployment

1.1 Download OpenStack RC File 

Ansible is used as the management tool to automate the creation and deploy virtual machines and set up software environment. Ansible rely on using Openstack to connect to Cloud platform, before we start we have to install Ansible and Openstack SDK on our local machine. After that an openstack file that enable us create VMS at Melbourne Research Cloud should be download and put it into the folder of ansible playbook.

picture1: OpenStack RC File
1.2 Instance creation

In this part, how to use Ansible to create VMs on Melbourne Research Cloud will be introduced. The structure of instance_create is this:

In the "nectar.yaml" the variables are defined, the instances, security groups and volumes will are defined in this file. "unimelb-comp90024-group-78-openrc.sh" is an OpenStack RC File that is used to connect to Melbourne Research Cloud. In "roles" folder, tasks are defined that include create images, instances, volumes and security groups. After that all roles will be executed in "create.yaml". The detail of "create.yaml" is shown below. These ymal code shows in our computer we use Ansible and OpenStack to do common settings, create security groups and instances, in my account there is four volumes, so I do not create extra volume. 


- hosts: localhost
vars_files:
- host_vars/nectar.yaml
  gather_facts: true

  roles:
     - role: openstack-common
     - role: openstack-security-group
     - role: openstack-instances


After "instance_create" is finished, a shell script "create.sh" will be utilized to execute this ansible playbook, there only one line in "create.sh". When run this shell a password should be used, this password can obtain from "Settings" in picture one. The process is that "settings -> Reset Password ".
. ./pt-45520-openrc.sh; ansible-playbook --ask-become-pass create.yaml


1.3 Instance settings

After four instances are generated, we need use them for different purposes, in this application two of the will be used to build a couchdb clusters by using docker, one of them is a harvester to obtain tweeters form TweetAPI, another one is a webserver to run our web application. Their groups and IP address is declared in a inventory file and these IP address play a key role for further settings.
			
[DBServers]
172.26.38.86
172.26.38.180

[Harvester]
172.26.38.194

[WebServer]
172.26.38.145


1.3.1 Building Couchdb cluster

There are there steps to enable a couchdb cluster by ansible playbook. First, seting the system environment and install dependent tools. Second, using docker to install couchdb and do basic settings which include docker proxy settings and couchdb settings. Finally, enable the cluster on one coordinate couchdb. All details will be included in docker-couchdb-playbook and this can be found in "https://github.com/orgs/comp90024-team78/dashboard". 

Before the start of setting, there are three important issues should be noticed. First, the security groups that declare the port range can be used. Second, ensure the firewall of instances are shut down. Finally, ensure all nodes can communicate with each other. Then, the automatic deployment of cluster can start.

Two build a cluster, two roles are necessary. Under folder “roles”, couchdb-container responsible for build container on remote host and the build-cluster connect these coucbdb nodes. According to the CouchDB official website, we need to biuld a container on each node, but only enable the cluster once in the coordinate node. This is the content of system-setting.yaml.
 #this script runs on the instances defined under inventory.ini
- hosts: DBServers
  gather_facts: true

  # create couchdb nodes
  roles:
    - role: mount-volumes
    - role: docker-setting
    - role: couchdb-container

# enable cluster on the coordinatenode
- hosts: DBServers[0]
  gather_facts: true

  roles:
    - role: build-cluster


After that, a shell commond shoud be used to run this yaml file and a couchdb cluster is enabled. Then we can use “curl http://your-ip-address:5984/_membership “ to check the cluster.
#!/usr/bin/env bash
cat ~/.ssh/team.pem
. ./unimelb-comp90024-group-78-openrc.sh; ansible-playbook -i ./inventory/inventory.ini -u ubuntu --key-file=~/.ssh/team.pem system-setting.yaml


1.3.2 Deploy Harvester and WebServer
They are simillar to building a couchdb cluster. A volume attached to harvester shoud be mounted to a specific directory and python3, couchdb module and all dependncies are installed. Then copy the tweet spider to harvester and run it in the background.


