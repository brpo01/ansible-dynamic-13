# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

This project shows an implementation of dynamic assignments using the `include` module and what makes it different from static assignments. As we've seen from the previous [project](https://github.com/brpo01/ansible-static-12/blob/master/ansible-static-12.md), static assignments make use of the `import` module and dynamic assignments mae use of the `include` module. What makes these two types of assignments different? well, let's look into it.

In Static assignments, all statements are pre-processed at the time playbooks are parsed. Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static. On the oher hand Dynamic assignments processes all statements only during execution of the playbook, which means that after the statements are parsed, any changes to the statements encountered during execution will be used. This is what makes it dynamic.

# INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE
- In the previous [project]((https://github.com/brpo01/ansible-static-12/blob/master/ansible-static-12.md) GitHub repository start a new branch and call it dynamic-assignments.

- Create a new folder known as dynamic-assignments, this folder will store the playboo that'll dynamically include the environmental variables in our set of tasks. Within that folder, create a file and call it env-vars.yml

```
$ sudo mkdir dynamic-assignments
$ sudo touch env-vars.yml
```
- In the previous project, we also created different inventory files that store details/info about the target servers in different environments i.e (dev, staging, uat, prod). For dynamic assignments we'll be adding a new concept called variables, these variables will store info about certain tasks that need to be executed based on the environment where it is being run. So for example, if the project is being run in the `staging` environment, the playbook will pick out all the env. variables based on the environment and run the playbook on the target servers specified in the inventory file. 

- Create a new folder called `env-vars` and a file for each of the environments.

- The layout should look like this:

```
$ sudo mkdir env-vars
$ sudo touch dev.yml uat.yml staging.yml prod.yml
```
- The layout should look like this:

```
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml

```

- Paste the following instruction into the env-vars.yml file. The playbook below basically collates the variables that are found from the env. specific files and run the tasks. It'll loop through all the env. files to look for variables and match it with the env. specific inventory file, pick out all the target servers and run the tasks on them.

![2](https://user-images.githubusercontent.com/47898882/130084408-86d2a080-fb91-4776-bb05-6f5874487fe4.JPG)

- There are some things to take note of in this playbook

1. We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are: include_role, include_tasks, include_vars

2. We made use of a special variables ` playbook_dir` and `inventory_file`. `playbook_dir` will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. `inventory_file`on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. `with_first_found` implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

- Update the site.yml file to make use of the dynamic assignments

![3](https://user-images.githubusercontent.com/47898882/130086691-60205268-fb49-41a0-b034-68598cd049f7.JPG)


# Community Roles
- Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users, and also install the apache & nginx webservices as we'll be configuring a loadbalancer to distribute requests between the web-servers. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

- The roles we are going to be installing are mysql, apache & nginx, we're going to be using roles created by geerlingguy. Use the `ansible-galaxy` command to download the role on your Bastion Host server

```
$ ansible-galaxy install geerlingguy.mysql
$ ansible-galaxy install geerlingguy.apache
$ ansible-galaxy install geerlingguy.nginx
```
- Rename the roles for easier identification.

```
$ mv geerlingguy.mysql/ mysql
$ mv geerlingguy.apache/ apache
$ mv geerlingguy.nginx/ nginx
```
- After installation, to understand how to configure your roles properly, study the README.md file of the specific role you're working with.

- To configure mysql role, go to the defaults/main.yml directory and add the name of the database and user. In this case the name of our database is `tooling` and the user is `webaccess`

![4](https://user-images.githubusercontent.com/47898882/130089218-91199a54-c48d-4ec8-bf19-03ab611a3350.JPG)

- To configure the apache role, go to the `defaults/main.yml` directory and add the ip-addresses of the webservers for the load_balancer configuration. Secondly, go into the `templates/vhost.conf.j2` directory and add the loadbalancer configuration within the virtualhost tag. 

*defaults/main.yml*

![5](https://user-images.githubusercontent.com/47898882/130089646-d1218ad9-eaa0-47aa-bf9b-2a5e0fe2331f.JPG)

*templates/vhost.conf.j2*

![6](https://user-images.githubusercontent.com/47898882/130090545-cfc8ca2e-7289-44bd-9fed-8c0a6230441a.JPG)

- To configure the nginx role, go to the `defaults/main.yml` directory and add the ip-addresses of the webservers for the load_balancer configuration. Set the webservers hostnames within the tasks/main.yml directory. Lastly, add the loadbalancer configuration within the `templates/nginx.conf.j2`

*defaults/main.yml*

![8](https://user-images.githubusercontent.com/47898882/130092099-eb8cb24b-4101-4d81-a24f-a8bbfebc91cb.JPG)

*tasks/main.yml*

![9](https://user-images.githubusercontent.com/47898882/130092111-61bcb5b5-1a2e-40b1-8e79-d427443875b8.JPG)

*templates/nginx.conf.j2*

![7](https://user-images.githubusercontent.com/47898882/130092718-0e8e4dbd-95a9-4353-a785-e528416243ae.JPG)

- To practicalize the dynamic assignment concept, we have to add environmental variables to our roles. To perform this task, go the the default/main.yml file of the apache and nginx roles, create a variable, and set the variables to false. In this case we are telling both roles, to set these variables to false by default, then we can decide to set it to true in the env-vars folder based on the environment(dev, uat, staging, prod) we are running the tasks on. This explains why this method is nown as dynamic assignments

*defaults/main.yml*

![10](https://user-images.githubusercontent.com/47898882/130093789-6b29d0d3-5331-4633-a11f-cb4ab64b62e8.JPG)

*defaults/main.yml*

![11](https://user-images.githubusercontent.com/47898882/130093803-25f5485e-f09a-43fe-b733-cb2dd103c4d6.JPG)

- Create a new file within the static-assignments folder and add the loadbalancer.yml file. From the playbook you can see that it'll run the particular role that has both `enable_lb` & `load_balancer_is_required` set to true.

![12](https://user-images.githubusercontent.com/47898882/130094865-33825ea9-ad09-42d9-9035-e49c1b7635fd.JPG)

- Now let us use the uat environment to test this configuration. Go to the env-vars/uat.yml directory and set both variables to true

![13](https://user-images.githubusercontent.com/47898882/130095697-034cbc13-fa67-4a9b-b815-d9cf586d2769.JPG)

- Run your playbook in the uat environment using the command below. It should run successfully, if you configured properly.

```
$ ansible-playbook -i inventory/uat.yml playbook/site.yml
```
![1](https://user-images.githubusercontent.com/47898882/130096037-9185348e-7f67-4dde-86ea-0dad0bc3fdc2.JPG)

- The same must work with nginx LB, so you can switch it by setting respective environmental variable to true and other to false. To test this, you can update inventory for each environment and run Ansible against each environment. 

### Congratulations, you have handled configuration management using ansible dynamic assignments (include) and community roles.

