# DevOps 360°

DevOps 360° is an introduction to automation with Ansible. For more details about the project, please check: http://slides.com/floriandambrine/devops360

## 0. Setup the Ansible controller

```
sudo yum install python-pip git
sudo pip install -U pip
pip install ansible==2.4.3.0
```

#### 0.1. Install Ansible roles dependencies from Galaxy

```
cd ansible/
ansible-galaxy install -r requirements.yml
```
## 1. Stage 1 - Drinks

##### 1.1. Find out which vars to override from [ansible-role-libvirt/defaults/vars.yml](https://github.com/Lowess/ansible-role-libvirt/blob/master/defaults/main.yml) so that you can use the module on your own server

The role `libvirt` allows you to start VMs using KVM. You will have to override certain variables so that the role can work on your own server.

> :interrobang: Determine which variables you should override from [defaults/vars.yml](https://github.com/Lowess/ansible-role-libvirt/blob/master/defaults/main.yml) in order to make the module working on your server.

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

> :interrobang: For now, the goal is to install and configure the version `v1.0.0` of the webapp and the database.

> :round_pushpin: As you can see, both of you need to deploy the devops-360-webapp repository, it might be a good idea to coordinate with your peer and create a `role` for that so that you do not do the same job twice.

##### 3.1. `webservers.yml`

* Checkout the application requirements that the developers have left for you in [2. Web application](https://github.com/Lowess/devops-360-webapp#2-web-application)

* You will find all the details of the application endpoints if you need [Application endpoints](https://github.com/Lowess/devops-360-webapp#22-application-endpoints)

> :interrobang: Think about what roles you should create to automate the BeerBattle webapp and implement them.

> :round_pushpin: Here are some useful links for you that you may need [How to Serve Flask applications with UWSGI](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-14-04), [Python Virtualenv](https://virtualenv.pypa.io/en/stable/), [Quickstart for Python WSGI applications](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html#quickstart-for-python-wsgi-applications)

##### 3.2. `databases.yml`

* Checkout the database requirements that the developers have left for you in [1. Mysql Database](https://github.com/Lowess/devops-360-webapp#1-mysql-database)

> :interrobang: Think about what roles you should create to automate the BeerBattle database and implement them.

> :round_pushpin: Here are some useful links for you that you may need [Ansible Mysql modules](http://docs.ansible.com/ansible/list_of_database_modules.html#mysql), [A Basic Mysql Tutorial](https://www.digitalocean.com/community/tutorials/a-basic-mysql-tutorial), [MySQL Utilities ~/.my.cnf](http://stackoverflow.com/questions/16299603/mysql-utilities-my-cnf-option-file).

##### 3.3. Note about accessing `00.webserver` on port 80

If you can not connect to `00.webserver.domXYZ.u13.org`, You will have to install Nginx and configure it as a reverse proxy to send traffic to `00.webserver` on the Ansible controller, the configuration should look like:

```
upstream devops-360-proxy {
    server 00.webserver.domXYZ.u13.org;
}

server {
    listen *:80;
    server_name _;

    access_log /var/log/nginx/devops-360-proxy-access.log;
    error_log /var/log/nginx/devops-360-proxy-error.log;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://devops-360-proxy;
    }
}
```


## 4. Stage 4 - Cheese plate

In this section we will work on scalability and redundancy. Let's add an extra webserver and a loadbalancer to the infrastructure:
  * One more webserver (VM name = `01.webserver` | ip = `172.16.XYZ.61`)
  * One load balancer (VM name = `00.loadbalancer` | ip = `172.16.XYZ.50`)

> :interrobang: The webserver part should be really quick to do as this server is the same as `00.webserver`. For `00.loadbalancer` you can install Nginx and use Nginx to do loadbalancing over your two webservers ([Using nginx as HTTP load balancer](http://nginx.org/en/docs/http/load_balancing.html)). If you already built an `nginx` role, you can reuse it, otherwise it might be time to refactor some code :smirk: ...

Everything is working? It's time for the desert then!

## 5. Stage 5 - Desert

Let's do a release process.

If we sum-up, right now our infrastructure is made of:
* `00.loadbalancer` : Clients entrypoint
* `00.webserver` : Web server part of the loadbalancer serving the webapp
* `01.webserver` : Web server part of the loadbalancer serving the webapp
* `00.webserver` : Web server part of the loadbalancer serving the webapp
* `00.mysql` : The database used by the webapp

Let's release `v2.0.0` on the webserver `00.webserver`. For this deployment we will adopt a [Canary release](https://martinfowler.com/bliki/CanaryRelease.html) process:

![Image Canary release](https://martinfowler.com/bliki/images/canaryRelease/canary-release-2.png)

> :interrobang: During this deployment, the database `00.mysql` will also have to upgrade the database schemas. Make sure your automation runs on `00.mysql` and `00.webserver` but make sure to leave `01.webserver` intact! Once the release is done, you should have both `v1.0.0` and `v2.0.0` available through the loadbalancer `00.loadbalancer`. Update whatever variables you need to do this release process, you can also use `--extra-vars` argument with your `play` command.

If you can not connect to `00.loadbalancer.domXYZ.u13.org`, You will have to install Nginx and configure it as a reverse proxy to send traffic to `00.loadbalancer` on the Ansible controller, the configuration should look like:

```
upstream devops-360-proxy {
    server 00.loadbalancer.domXYZ.u13.org;
}

server {
    listen *:80;
    server_name _;

    access_log /var/log/nginx/devops-360-proxy-access.log;
    error_log /var/log/nginx/devops-360-proxy-error.log;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://devops-360-proxy;
    }
}
```

> :interrobang: Is the `v2.0.0` running properly ? What should we do we do now ?

> :interrobang: Depending on how went the release, you should be fully running on `v1.0.0` or on `v2.0.0`.

A full-course meal is never done without a digestive...

## 6. Stage 6 - Digestive

Let's do a [Blue/Green](https://martinfowler.com/bliki/BlueGreenDeployment.html) deployment on the infrastructure.

Blue/Green deployment means that you should duplicate your stack like:
![Image of Blue/Green Infrastructure](https://martinfowler.com/bliki/images/blueGreenDeployment/blue_green_deployments.png)

Here is how your inventory should look like:

```
###############################################################################
### Blue Stack
###############################################################################

[webservers-blue]
[00:01].webserver.blue.dom110.u13.org

[databases-blue]
00.mysql.blue.dom110.u13.org

[loadbalancers-blue]
00.loadbalancer.blue.dom110.u13.org

[blue:children]
webservers-blue
databases-blue
loadbalancers-blue

###############################################################################
### Green Stack
###############################################################################

[webservers-green]
[10:11].webserver.green.dom110.u13.org

[databases-green]
10.mysql.green.dom110.u13.org

[loadbalancers-green]
10.loadbalancer.green.dom110.u13.org

[green:children]
webservers-green
databases-green
loadbalancers-green

###############################################################################
### General section
###############################################################################

[webservers:children]
webservers-blue
webservers-green

[databases:children]
databases-blue
databases-green

[loadbalancers:children]
loadbalancers-blue
loadbalancers-green
```

Create a new inventory like `inventories/blue-green` and drop the above `host` file in there.

If needed: On the ansible controller, drop the following configuration in `/etc/nginx/sites-available/proxy` (Remember to create the symlink in `/etc/nginx/sites-enabled/proxy` and make sure the `/etc/nginx/nginx.conf` has the `include /etc/nginx/sites-enabled/*;` statement in the `http` block):

```
resolver 172.16.XYZ.254;

server {
    listen *:80;
    server_name _;

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

Made with ♥ for teaching people
