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
  <img src="/screenshots/aws_vault_creds.png" width="950" title="Envrypted data">
  <br>
  <em>Fig 3.: Encrypted Credentials </em>
</p>


