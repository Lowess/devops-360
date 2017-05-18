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

##### 1.2. Create an alias to make your life easier

> :interrobang: In order to keep the command line light, we will create a bash `alias` (add this line the your `~/.bashrc`) with the following command:

```
alias play="ansible-playbook -i inventories/vms"
```

To make the alias active, either logout/login or run `source ~/.bashrc`.

From now, you will use `play` instead of `ansible-playbook`. Keep in mind what is happening behind the scene when you use the alias.

##### 1.3. Test the `libvirt` role

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

##### 2.1. Get ready with the infrastructure

Now that you have a configured `libvirt` role, we will start working on a first stage of the project [**BeerBattle** alias devops-360-webapp](https://github.com/Lowess/devops-360-webapp).

Let's begin with a simple infrastructure for now:
  * One webserver (VM name = `00.webserver` | ip = `172.16.XYZ.60`)
  * One database (VM name = `00.mysql` | ip = `172.16.XYZ.70`)

> :interrobang: Let's override the `libvirt_vms` variables in our `group_vars` to define new VMs.

Run the `vm-create.yml` playbook and make sure your infrastructure comes up properly.

##### 2.2. Break the **BeerBattle** project into `roles`

> :interrobang: Start thinking about ansible `roles`. How can you break the [**BeerBattle**](https://github.com/Lowess/devops-360-webapp) project into roles ? Think about DevOps best practices (monitoring tools, application users, pieces of automation shared between the database and the webserver).

Once you have your roles defined, init them with the command:
```
cd <root>/ansible/roles
ansible-galaxy init <rolename>
```

##### 2.3. Work on common role(s) together

Start building the common role(s) with your peer, this role(s) will then be added to `webservers.yml` and `databases.yml`

Once you think you have the common parts, let's order the main course!

## 3. Stage 3 - Main course

For this stage, we will split the two-person team into two. One of you will work on `webservers.yml` and specific web roles and the other one will work on `databases.yml` and the related roles.

##### 3.1. `webservers.yml`

* Checkout the application requirements that the developers have left for you in [2. Web application](https://github.com/Lowess/devops-360-webapp#2-web-application)

> :interrobang: Think about what roles you should create to automate the BeerBattle webapp and implement them.

##### 3.2. `databases.yml`

* Checkout the database requirements that the developers have left for you in [1. Mysql Database](https://github.com/Lowess/devops-360-webapp#1-mysql-database)

> :interrobang: Think about what roles you should create to automate the BeerBattle database and implement them.

## 4. Stage 4 - Cheese plate

In this section we will work on scalability and redundancy. Let's add an extra webserver and a loadbalancer to the infrastructure:
  * One more webserver (VM name = `01.webserver` | ip = `172.16.XYZ.61`)
  * One load balancer (VM name = `00.loadbalancer` | ip = `172.16.XYZ.50`)


## 5. Stage 5 - Desert

Let's do a release process.

## 6. Stage 6 - Digestive

Let's do a blue/green deployment on the infrastructure.

```
resolver 172.16.XYZ.254;

server {
    listen *:80;
    server_name florian.u13.org;

    access_log /var/log/nginx/beerbattle-loadbalancer-access.log;
    error_log /var/log/nginx/beerbattle-loadbalancer-error.log;

    set $upstream http://beerbattle.domXYZ.u13.org;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass $upstream;
    }
}
```