# Automated Server Provisioning and Application Deployment using Ansible on AWS
The resource provisioning in Amazon Web Services utilising various ansible modules. The automation of provisioning and web server configuration on AWS with the help of ec2 dynamic inventory. The instance provisioning is not recommened with the ansible, since ansible does not stores the instance state.

**Project Flow Diagram**

<p align="center">
  <img src="/screenshots/infra_flow.jpg" width="950" title="Infra Diragram">
  <br>
  <em>Fig 1.: Project Flow Diagram </em>
</p>


## Scope of Project
- Create EC2 Instance
- Configure Web Server using EC2 dynamic inventory

Two different playbooks are created for both the tasks and invoked using `import_playbook` ansible module.

>Note:
>
> Kindly follow the usage instructions to run the play without any errors. The different plays are explained individually in the project just for reference and better understanding.

### Pre-requisites (Controller_Node)

- **Operating System:** Redhat Enterprise Linux 7 or above
- **Softwares:** Ansible v2.9.9, Python v3.6.8
- **AWS:** Programmatic Access Credentials i.e access_key and secret_key


### Configuration of Ansible

Ansible Configuration Tool is configured with the custom parameters in configuration file i.e *ansible.cfg*. The path for inventory has been upadted to the dynamic inventory script ec2.py. The AWS Credentials are configured in `ec2.ini` for accessing the resources in Amazon Web Services. The file can be used to customise the dynamic inventory script for EC2 instances.

The AWS credentials i.e; access_key and secret_key is to be assigned in `ec2.ini` at the end of file:
```
[credentials]
aws_access_key_id = XXXXXXXXXXXXXXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXXXXX
```

The shebang in the dynamic inventory is changed to use python3 interpretor instead of python2
```sh
vim ansible_configuration/ansible_hosts/ec2.py
#!/usr/bin/python3
```

**Installation of Required python libraries**

The python `boto` library is required for the dynamic inventory to interact with the AWS SDK on the *Ansible Controller Node*. The boto library is installed using pip3 installer.

```sh
yum install -y python3-pip
pip3 install -U boto
```

**Verify the dynamic inventory**

The ansible should be able to list out all *running* Amazon EC2 instances using `ansible all --list-all`.

<p align="center">
  <img src="/screenshots/ansible_inventory_list.png" width="950" title="Hosts">
  <br>
  <em>Fig 2.: Ansible Hosts List</em>
</p>

The ansible configuration and dynamic inventories used in project are present in `ansible_configuration` directory for reference.


### Provisioning of Amazon EC2 Server 

For provisioning of resources on Amazon Web Services, the programmatic credentials are configured and passed to the the tasks in playbook i.e *playbooks/ec2_instance_create.yml*. The credentials are encrypted using ansible-vault using vault-password file `vault_pass`.

**Ansible-Vault file **

Ansible Vault is a feature of ansible that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext in playbooks or roles. The AWS Credentials are stored in the vault file when used in tasks in playbook. The vault password is stored in `vault_pass` file. The data in aws_creds.yml file i.e access_key and secret_key are encrypted using ansible vault.

```
ansible-vault create --vault-id prod@vault_pass aws_creds.yml
```

Raw plain text data before encryption of aws_creds.yml
```
aws_creds.yml
---
access_key: AWSDEFSFSEFSDAWDA
secret_key: askdgaaiugff093lkkna+asdasd
```

<p align="center">
  <img src="/screenshots/aws_vault_creds.png" width="950" title="Encrypted data">
  <br>
  <em>Fig 3.: Encrypted Credentials </em>
</p>


**Tasks in the play**

AWS_credentials are imported into the play using `import_vars` ansible module. The `ec2_key` ansible module is used to create Instance key with name *ec2_instance_key*. The `curl` module is used to fetch public IP of the controller node to  allow SSH traffic only from the controller node. The `ec2_group` ansible module is used to provision the firewall for the instance to access the application on 80 port worldwide and 22 port only from controller node. The `ec2` ansible module is used to provision the EC2 instance using the provided aws_creds. The `wait_for` module to wait for the instance to be ready for recieving SSH connection.

The inventory is refreshed using `meta` module to update inventory using `dynamic inventory` with the launched resource. The handler is used to store the key as soon as a new key is created.

The ansible will create instances on every play run since it does not tracks the instance state; so to prevent the  ansible to launch the instance on every play, the `tags` are used i.e instance_launch. The tag `instance_launch` is passed only when a new instance is to be launched.

```
ansible-playbook ec2_instance_create.yml --vault-id prod@vault_pass --tags instance_launch
```


<p align="center">
  <img src="/screenshots/ansible_ec2_resource.png" width="950" title="Play">
  <br>
  <em>Fig 4.: Ansible Play for EC2 resource creation </em>
</p>

The `ec2_instance_create.yml` playbook is present in the directory *playbooks* for reference.


### Web Server Configuration

The web_server ansible role is used in `web_server_configuration.yml` playbook to perform various tasks for cconfiguration for web server and application deployment in the newly launched EC2 instance. The Apache Web Server Configuration and code_deployment is tested using `post_tasks` in the play.

The instance key generated in the previous play i.e; *ec2_instance_key.pem* is being used in the play to connect with the instance using ssh.

**Tasks in the play**

The variables are defined in the `vars/main.yml` inside the web_server role. The required packages are installed in the instance using `yum` module. The application code directory is created using `file` module. The code is deployed from the github using `git` module. The SELINUX is disabled premanently to ease the connectivity of the application. Since only one *Listen* directive with port number 80 can be used in apache configuration, so the replace module is used to remove the Listen directive if the variable `http_port` is assigned with 80 port number. The apache server is started using `systemd` module in case the server is in stopped state.


The `post_tasks` defined in `tests/test.yml` in the module is used to test the cconfiguration of apache server and deployment of application code. The play can be run with the help of below command if the inventory is updated.
`
```
ansible-playbook -l ec2 web_server_configure.yml
```

<p align="center">
  <img src="/screenshots/ec2_play.png" width="950" title="Play">
  <br>
  <em>Fig 4.: Web Server Configuration and Testing </em>
</p>


# Usage Instructions

The `pre-requisites` should be satisfied and the `vault-file` should be created, the `dynamic inventory` *ec2.py and ec2.ini* should be configured using the above mentioned steps. Then

1. Clone this repository
2. Change the working directory to `ansible_configuration`
3. Copy your credentials in credentials directory with same name or need to configure the file names in the variables
4. Execute the playbook `main.yml` present in playbooks directory

```
ansible-playbook ../playbooks/main.yml --vault-id prod@../credentials/vault_pass
```

The `main.yml` imports both the plays i.e ec2_instance_create.yml and web_server_configure.yml and executes it one by one. 

## Variables

The list of variables used in the plays

**EC2 Instance Creation Play**

| Name | Description | Default | Required |
|------|-------------|:-----:|:-----:|
| region_name | Default Region Name for Instance Launch | `ap-south-1` | yes |
| instance_type | Instance Configuration i.e CPU and memory | `t2.micro` | yes |
| instance_ami | AMI-Id of the Image | `ami-052c08d70def0ac62` | yes |
| number_of_instances | Number of instance to launch | `1` | yes |
| subnet_id | Subnet Id determines the Availability Zone of Instance | `` | yes |
| vpc_id | Network for Instance Launch | `` | yes |


**Web Server Configure Play**

| Name | Description | Default | Required |
|------|-------------|:-----:|:-----:|
| http_port | Web Server Port | `80` | yes |
| packages_list | Packages to installed on Instance | `[ 'httpd', 'git' ]` | yes |
| application_root_directory | Application Deployment Directory | `/var/www/html/web` | yes |
| tmp_apache_configuration_file | Temporary Configuration file | `/opt/web.conf` | yes |
| apache_configuration_file | Apache Configuration File | `/etc/httpd/conf.d/web.conf` | yes |
| apache_mods_enabled | Directory that conatins Apaache Modules configuration files | `/etc/httpd/conf.modules.d` | yes |
| code_repo | Application Code | `https://github.com/riteshsoni10/demo_website.git` | yes |


## ScreenShots

**1. Instance Key**

<p align="center">
  <img src="/screenshots/aws_ec2_key.png" width="950" title="Key">
  <br>
  <em>Fig 5.: Instance Key </em>
</p>


**2. Security Group**

<p align="center">
  <img src="/screenshots/aws_ec2_security_group.png" width="950" title="Security Group">
  <br>
  <em>Fig 6.: AWS EC2 Security Group </em>
</p>


**3. AWS EC2 Instance**

<p align="center">
  <img src="/screenshots/aws_ec2_instance.png" width="950" title="EC2 Instance">
  <br>
  <em>Fig 7.: AWS EC2 instance </em>
</p>


> **Source**: LinuxWorld Informatics Pvt Ltd. Jaipur
>
> **Under the Guidance of** : [Vimal Daga](https://in.linkedin.com/in/vimaldaga)

