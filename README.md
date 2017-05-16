# DevOps 360°

DevOps 360° is an introduction to automation with Ansible. For more details about the project, please check: http://slides.com/floriandambrine/devops360

## 0. Setup the Ansible controller

```
sudo yum install python-pip git
sudo pip install -U pip
pip install ansible==2.3.0.0
```

## 1. Stage 1 - Drinks

##### 1.1. Override [libvirt/defaults/vars.yml](ansible/roles/libvirt/defaults/main.yml) so that you can use the module on your own server

The role `libvirt` allows you to start VMs using KVM. You will have to override certain variables so that the role can work on your own server.

> :interrobang: Determine which variables you should override from [defaults/vars.yml](ansible/roles/libvirt/defaults/main.yml) in order to make the module working on your server.

> :interrobang: Once you have identified the variables you need to override, create an inventory folder called `vms` under `<root>/ansible/inventories/` and add an empty `hosts` file under `vms`. Create `group_vars/all/libvirt` and add `group_vars/all/libvirt/vars.yml` file with your variable overrides.

> :interrobang: In order to keep the command line light, we will create a bash `alias` (add this line the your `~/.bashrc`) with the following command:

```
alias play="ansible-playbook -i inventories/vms"
```

To make the alias active, either logout/login or run `source ~/.bashrc`.

From now, you will use `play` instead of `ansible-playbook`. Keep in mind what is happening behind the scene when you use the alias.

At this stage, you are ready to test if you did well so far :sweat_smile:, Let's try to run `vm-create.yml` playbook:

```
play vm-create.yml

### If you want you can run virsh list to make sure the VM is real
virsh list
```

If the playbook completes successfully, try to SSH into your VM with:

```
ssh root@ansible-test.domXYZ.u13.or
```

If that works, let's destroy this ansible-test VM with:

```
play vm-delete.yml

### Again, you can run virsh list to make sure the VM is gone
virsh list
```

...and let's order appetizers!

## 2. Stage 2 - Appetizers
  TODO
## 3. Stage 3 - Main course
  TODO
## 4. Stage 4 - Cheese plate
  TODO
## 5. Stage 5 - Desert
  TODO
## 6. Stage 6 - Digestive
  TODO
