# Ansible_deployment

# Download OpenStack RC File 
Ansible is used as the management tool to automate the creation and deploy virtual machines and set up software environment. 
Ansible rely on using Openstack to connect to Cloud platform, before we start we have to install Ansible and Openstack SDK on 
our local machine. After that an openstack file that enable us create VMS at Melbourne Research Cloud should be download and 
put it into the folder of ansible playbook.

#  Instance creation
In the "nectar.yaml" the variables are defined, the instances, security groups and volumes will are defined in this file. 
"unimelb-comp90024-group-78-openrc.sh" is an OpenStack RC File that is used to connect to Melbourne Research Cloud. In "roles" 
folder, tasks are defined that include create images, instances, volumes and security groups. After that all roles will be 
executed in "create.yaml". The detail of "create.yaml" is shown below. These ymal code shows in our computer we use Ansible 
and OpenStack to do common settings, create security groups and instances, in my account there is four volumes, so I do not 
create extra volume. 

After "instance_create" is finished, a shell script "create.sh" will be utilized to execute this ansible playbook, there only 
one line in "create.sh". When run this shell a password should be used, this password can obtain from "Settings" in picture 
one. The process is that "settings -> Reset Password ". 

# Instance settings
After four instances are generated, we need use them for different purposes, in this application two of the will be used to 
build a couchdb clusters by using docker, one of them is a harvester to obtain tweeters form TweetAPI, another one is a 
webserver to run our web application. Their groups and IP address is declared in a inventory.ini file and these IP address play a 
key role for further settings.			
# Building Couchdb cluster, WebServer and Harvester
In docker-couchdb-playbook there is a file named "system-setting.yaml" that include all tasks need to be played i a VM. You just need to run "setting.sh", everything will be done.
