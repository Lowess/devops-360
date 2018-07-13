# Playbooks Overview

### 1. infra-create.yml

> Spin up VMs and and run automations against databases and webservers. Once the playbook completes, the infrastructure is fully up and running.

```
ansible-playbook -i inventories/vms infra-create.yml
```

### 2. infra-delete.yml

> Delete the whole infrasture.

```
ansible-playbook -i inventories/vms infra-delete.yml
```

### 3. vm-create.yml

> Create all the VMs used for the infrastructure.

```
ansible-playbook -i inventories/vms vm-create.yml
```

### 4. vm-delete.yml

> Delete all the VMs used for the infrastructure.

```
ansible-playbook -i inventories/vms vm-delete.yml
```

### 5 databases.yml

> Configure the databases.

```
ansible-playbook -i inventories/vms databases.yml
```

### 6 webservers.yml

> Configure the webservers.

```
ansible-playbook -i inventories/vms webservers.yml
```


### 7 ngrok.yml

> Install [Ngrok](https://ngrok.com/) to open secure introspectable tunnels to localhost for easily accessing your Web UIs.


```
ansible-playbook -i inventories/vms ngrok.yml
```
