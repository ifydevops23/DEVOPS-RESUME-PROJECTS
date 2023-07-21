# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES<br>

**IMPORTANT NOTICE:** Ansible is an actively developing software project, so you are encouraged to visit Ansible Documentation for the latest updates on modules and their usage.<br>

Now you will continue configuring your UAT servers learning and practicing new Ansible concepts and modules.
In this project we will introduce dynamic assignments by using include module.<br>

Now you may be wondering, what is the difference between static and dynamic assignments?
Well, from Project Ansible Static-Assignments, you can already tell that static assignments use import Ansible module. <br>
The module that enables dynamic assignments is **include**<br>
Hence,<br>
import = Static<br>
include = Dynamic<br>

When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements.<br>
This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.
On the other hand, when include module is used, all statements are processed only during execution of the playbook.<br>
Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.<br>

Take note that in most cases it is recommended to use static assignments for playbooks, because it is more reliable. With dynamic ones, it is hard to debug playbook problems due to its dynamic nature. However, you can use dynamic assignments for environment specific variables as we will be introducing in this project.
<br>


### INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE<br>

Introducing Dynamic Assignment Into Our structure<br>
In https://github.com/ifydevops23/ansible-config-mgt1 GitHub repository start a new branch and call it dynamic-assignments.<br>

![new-branch-dynamic-assignments](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/965f4540-b205-4bde-a8f4-e90ea8abbc39)


Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml.<br>

We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.
Your GitHub shall have following structure by now.<br>
Note: Depending on what method you used in the previous project you may have or not have roles folder in your GitHub repository – if you used ansible-galaxy, then roles directory was only created on your Jenkins-Ansible server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case – it is up to you.



```
├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
```

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.
For this reason, we will now create a folder to keep each environment’s variables file. <br>

Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

![create-env-vars-folder-and-files](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/ba7dcd6d-da9f-420a-832c-462e1cc3ce32)

Your layout should now look like this.<br>

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

![folder-structure-1](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/7a00a2a3-d654-4c42-b533-19f27ed7de38)


Now paste the instruction below into the env-vars.yml file.<br>

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

![env-vars-yml-edit](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/c58cb4a1-7061-4572-aa8b-c1ff188211d0)

Notice 3 things to notice here:<br>

We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:<br>
include_role<br>
include_tasks<br>
include_vars<br>
In the same version, variants of import were also introduces, such as:<br>
import_role<br>
import_tasks<br>

We made use of a special variables { playbook_dir }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. <br>

We are including the variables using a loop. <br>
with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.
<br>


### UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS<br>
Update site.yml with dynamic assignments<br>
Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

site.yml should now look like this.<br>
```
---
- name: import common file
  import_playbook: ../static-assignments/common.yml
  tags:
    - always

- name: include env-vars file
  import_playbook: ../dynamic-assignments/env-vars.yml
  tags:
    - always
```

## Community Roles<br>
Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. <br>
These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going. <br>

Download Mysql Ansible Role<br>
You can browse available community roles here<br>
We will be using a MySQL role developed by geerlingguy.<br>

Hint: To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ‘ansible-config-mgt’ directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory.<br>
In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it – we will be using Jenkins later for a better purpose.<br>
On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run

```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```

![git-branch-roles-feature](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/5193bd57-db8c-4d83-9218-ffb569df3336)

Inside roles directory create your new MySQL role with <br>
`ansible-galaxy install geerlingguy.mysql` <br>
and rename the folder to mysql <br>
`mv geerlingguy.mysql/ mysql`<br>

- Create db.yml file under static-assignments.

![added-geerling-mysql-role](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/ab9aeb1e-d47f-41ec-b118-f18f9b51b77b)


Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

![multiple_users_db](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/6ee5e19f-76e6-45ea-89e6-8905f4089055)

Now it is time to upload the changes into your GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```
Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.<br>


## LOAD BALANCER ROLES<br>
Load Balancer roles<br>

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:<br>
- Nginx<br>
- Apache<br>
With your experience on Ansible so far you can: Decide if you want to develop your own roles, or find available ones from the community<br>

![added-apache-nginx-role](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/8f15b09d-e3b2-4a2e-9fc9-4c19fbe4b03a)

![mv-nginx-apache](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/3d4cd3a3-3ddf-4134-b78a-42845e84b815)

Update both static-assignment and site.yml files to refer the roles<br>

**Edit Nginx Role to act as Load Balancer**<br>

![nginx-conf-1](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/f0ffb803-e1d5-478e-b13c-5f3ca6e25608)

![nginx-conf-2 - Copy](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/b2211cb4-4182-4b01-aab4-ab6773102761)

Important Hints:<br>
Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.
Declare a variable in defaults/main.yml file inside the Nginx and Apache roles.<br>
Name each variables <br>
enable_nginx_lb <br>
enable_apache_lb <br>
Declare another variable in both roles load_balancer_is_required and set its value to false as well<br>

Set both values to false like this enable_nginx_lb:false

![nginx-conf-2](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/1e57d85a-dcc6-4304-9f2b-4890d801781b)

and enable_apache_lb: false.<br>

![apache_config_defaults-main-yml](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/2ea3e1dc-fb7a-4606-bc03-8451d1639889)

Update both assignment and site.yml files respectively<br>

loadbalancers.yml file<br>

```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

site.yml file

```
     - name: Loadbalancers assignment
       hosts: lb
     - import_playbook: ../static-assignments/loadbalancers.yml
       when: load_balancer_is_required
```

Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.
You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file (Set to true in nginx/defaults/main.yml) <br>

The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.
```
enable_nginx_lb: true
load_balancer_is_required: true
```

To test this, you can update inventory for each environment and run Ansible against each environment.<br>

Congratulations!<br>

![play1_common_play](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/2a701f9e-040f-40de-84e1-91a6044354ad)

![play_db_3_edited](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/fc8ad55d-829f-4d9c-9e67-b53e972e2584)

![final_play_nginx_plus_summary](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/d4ef4fc3-a4f3-449e-b00a-d99e3a9cb687)

You have learned and practiced how to use Ansible configuration management tool to prepare UAT environment for Tooling web solution.<br>

### RESULTS <br>

- Servers Managed
  
![Instances_in_UAT_pls_lb_IP_duplicate - Copy](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/9a5e0687-a6b6-4eb8-9d6f-c3292ff0064d)

- DB config Success

![users_created_db](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/931e1837-445f-47b7-92d9-61ef7eb8d92a)

- Webservers Config Successful

![webserver_1](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/fade3b3e-6967-4aa6-8a7d-2f57a4249343)

- Load Balancer Config succesful (routes traffic to webservers 1 & 2)

![Instances_in_UAT_pls_lb_IP_duplicate](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/03af4033-ed32-44ae-b809-0f7017b72dff)


![loadb_pointing_to_webserver](https://github.com/ifydevops23/Ansible_Projects/assets/126971054/6dc8f329-f495-4a32-a4f5-48b265229252)


