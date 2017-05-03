# DevOps 360°

DevOps 360° is an introduction to automation with Ansible. For more details about the project, please check: http://slides.com/floriandambrine/devops360

## 1. Setup the Ansible controller

```
sudo yum install python-pip git
sudo pip install -U pip
pip install ansible==2.3.0.0
```

## 2. Playbook overview

### 2.1. infra-create.yml

> Spin up VMs and and run automations against databases and webservers. Once the playbook completes, the infrastructure is fully up and running.

```
ansible-playbook -i inventories/vms infra-create.yml
```

### 2.2. infra-delete.yml

> Delete the whole infrasture.

```
ansible-playbook -i inventories/vms infra-delete.yml
```

### 2.3. vm-create.yml

> Create all the VMs used for the infrastructure.

```
ansible-playbook -i inventories/vms vm-create.yml
```

### 2.4. vm-delete.yml

> Delete all the VMs used for the infrastructure.

```
ansible-playbook -i inventories/vms vm-delete.yml
```

### 2.5 databases.yml

> Configure the databases.
 
```
ansible-playbook -i inventories/vms databases.yml
```

### 2.5 webservers.yml

> Configure the webservers.

```
ansible-playbook -i inventories/vms webservers.yml
```
