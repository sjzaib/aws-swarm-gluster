# Ansible Playbook: aws-swarm-gluster

This ansible playbook deploys docker swarm cluster on AWS with distributed storage system (GlusterFS).

## Description:

Playbook will launch few Linux instances on AWS, installs docker and create swarm. All instances will be launched with an additional 5GB SSD EBS volume attached. These EBS disks will eventually form a gluster replicated volume. Gluster volume will be available at /swarm/volumes on each node. You can use this volume to launch docker services with shared volumes for high availibility. For example you can mount volume /swarm/volumes/html:/var/www/html for your apache2 serivce and content of /swarm/volumes/html will be available to all containers of this service, which allows you can easily scale up or down the service.

Sequence of tasks (in a nutshell): 

1. Create aws security groups and SSH Keys
2. Launch EC2 instaces
3. Install Docker
4. Create Docker Sarm
5. Install Gluster
6. Prepare EBS disk for Gluster
7. Create and mount Gluster volume


Note that, a fresh ssh key and a security group will be created for each cluster you launch. 
Cluster will be launched in your default VPC and subnet. 

Other defaults includes

- AWS region: us-east-2
- Instance type: t2.micro
- EBS volume size for gluster: 5 GB
- EBS volume type for gluster : General Purpose SSD
- OS AMI : ami-916f59f4 (Ubuntu x86_64 16.04 Official)
- Docker Version: 18.03.0~ce-0~ubuntu
- Gluster Version: 3.8
- Gluster Volume Mount point: /swarm/volumes
- No of Manger Nodes: 3
- No of Worker Nodes: 1

etc.

For complete list check group_vars/all.yml

## Requirements:

1. ansible >= 2.5 installed. 
2. python pip, boto and boto3 - but these can be installed through the playbook itself.

## Usage:

All you need to do is provide your AWS credentials and run the playbook.

### AWS Credentials:

Ansible requires python-boto library to call AWS API, and boto needs AWS credentials in order to perform all the functions. There are many ways to configure your AWS credentials.

USE ANY ONE of the following.

#### METHOD 1: Create a .boto file under your user home directory:

Then add the following:
```shell
[credentials]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>
```


#### METHOD 2: Create a .credentials file under user home ~/.aws/

Then add the following:
```shell
[default]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>
```

#### METHOD 3: Create a .config file under user home ~/aws/ :

Then add the following:
```shell
[default]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>
```


#### METHOD 4: Export manually in shell environment

```shell
export AWS_ACCESS_KEY=<your_access_key>
export AWS_SECRET_KEY=<your_secret_key>
```

Avoid editing aws_access_key and aws_secret_key variables directly in group_vars/all.yml.


### DEPLOY:

Once you have your aws credentials defined. The simplest way to launch cluster is to run


```shell
ansible-playbook playbook.yml -t deploy,pyset -e 'depid=01'
```

Tags allow to run/omit group of tasks from playbook. Tag "pyset" installs any required python dependencies your ansible controller host may be missing, such as pip, boto and boto3. You can omit this tag on your next run.  

IMPORTANT: It is highly recommended to mention "depid" (deployment id) environment variable at playbook runtime and use a unique numeric id for each launch. It enables us to launch/kill multiple mutually exclusive clusters. Each subsequent run of playbook with same depid will touch resources belonging to that depid only. 

If you don't mention depid on runtime, it will use default value for depid defined in group_var/all.yml file. This means everytime you run playbook without depid it will target the default id. 


### CUSTOMIZE:

All variable used by the playbook and associated roles are read from a centralized location group_vars/all.yml. File is self explanatory you can change different variable to achieve your purpose. For example

To change no. of nodes edit group_vars/all.yml


```shell
manager_count: <integer>
worker_count: <integer>
```

Default is a 4 node cluster with 3 managers and 1 worker. Managers also runs containers. So all your nodes will run containers but manager nodes also have additional responsibility to manage docker swarm.


To change aws region edit group_vars/all.yml
```shell
region: us-east-1
```
Default region set is us-east-2


For production use you may want to protect gluster EBS volume

```shell
delete_on_termination: False
```


Most significant variable is "depid". Value of this variable is attached to many resources this playbook creates. It helps to "identify" and link all artifacts of a particular deployment for idempotency.  

### REMOVE:

You can dismantle your entire cluster and all associated artifacts such as security groups and aws keys with

```shell
ansible-playbook nodes-rm.yml -t remove -e 'depid=01'
```

Except private key .pem file that is saved under key/ directory. This is to avoid accidental removal of critical information. If you must, remove it manually.

## TODO:

1. Create custom VPC and subnet and use it to launch our cluster.
2. Add support for other distros e.g. CentOS, Amazon Linux 
3. Add loadbalancer support

## LICENSE:

This is an MIT licensed project. Please check LICENSE file for more information.
