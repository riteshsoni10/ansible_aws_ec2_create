# Automated Server Provisioning and Application Deployment using Ansible on AWS
Automation of creation of EC2 Instance and Configuration of Apache Web Server using Ansible 


## Scope of Project
- Create EC2 Instance
- Configure Web Server using EC2 dynamic inventory

Two different playbooks are created for both the tasks and invoked using `import_playbook` ansible module.


### Pre-requisites

- **Operating System:** Redhat Enterprise Linux 7 or above
- **Softwares:** Ansible v2.9.9, Python v3.6.8
- **AWS:** Programmatic Access Credentials


### Configuration of Ansible

Ansible Configuration Tool is configured with the custom parameters in configuration file i.e *ansible.cfg*. The path for inventory has been upadted to the dynamic inventory script ec2.py. The AWS Credentials are configured in `ec2.ini` for accessing the resources in Amazon Web Services. The file can be used to customise the dynamic inventory script for EC2 instances.

The AWS credentials i.e; access_key and secret_key is to be assigned in ec2.ini at the end of file:
```
[credentials]
aws_access_key_id = XXXXXXXXXXXXXXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXXXXX
```


### Provisioning of Amazon EC2 Server 

For provisioning of resources on Amazon Web Services, the programmatic credentials are configured and passed to the the tasks in playbook i.e *playbooks/ec2_instance_create.yml*. The credentials are encrypted using ansible-vault using vault-password file `vault_pass`.
