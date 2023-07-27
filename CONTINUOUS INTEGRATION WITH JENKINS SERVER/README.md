# BUILD & DEPLOYMENT AUTOMATION WITH JENKINS SERVER

One of the ways to guarantee fast and repeatable deployments is the Automation of routine tasks.<br>
In this project, we are going to start automating part of our routine tasks with a free and open-source automation server – Jenkins.<br>
It is one of the most popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".<br>

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. 
Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository<br>
In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<username>/Tooling.git will be automatically be updated to the Tooling Website.<br>

Task<br>
Enhance the architecture prepared in Project 8 by adding a Jenkins server, and configuring a job to automatically deploy source code changes from Git to the NFS server.<br>
Here is what your updated architecture will look like upon completion of this project:<br>

![00_implemented](https://github.com/ifydevops23/CI_CD/assets/126971054/b62230a0-04cf-4517-9665-592deffd10bb)


## Step 1 – INSTALL THE JENKINS SERVER
- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins" <br>
- Install JDK (since Jenkins is a Java-based application)<br>
`sudo apt update`<br>
`sudo apt install default-jdk-headless`<br>

Install Jenkins<br>![1_install_jdk_headless](https://github.com/ifydevops23/CI_CD/assets/126971054/cb5826e4-7433-439f-8f10-30075c2d5602)

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`<br>
`sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`<br>
`sudo apt update`<br>
Add Key if asked for Public Key<br>
`sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA`<br>
![1_wget_debiamn_pkg](https://github.com/ifydevops23/CI_CD/assets/126971054/8f4a88f4-7441-40f5-92e5-df8fd458ab7f)

`sudo apt install jenkins`<br>
![1_install_jenkins](https://github.com/ifydevops23/CI_CD/assets/126971054/9c4dadd9-fe18-4e27-98ad-b96ed8d9960e)

Make sure Jenkins is up and running<br>
`sudo systemctl status jenkins`<br>
![1_status_jenkins](https://github.com/ifydevops23/CI_CD/assets/126971054/32d9b571-25ab-4006-bce0-0f2b9e969584)

By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 security group.<br>
![security_group](https://github.com/ifydevops23/CI_CD/assets/126971054/a3c37fb1-8474-49eb-9a92-9f6374877386)

Perform initial Jenkins setup.<br>
From your browser access 
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```

![1_login_admin_pass](https://github.com/ifydevops23/CI_CD/assets/126971054/3d114edb-1e71-4b31-bb84-d22c72d8dfb5)

You will be prompted to provide a default admin password.Retrieve it from your server: <br>
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![1_admin_password](https://github.com/ifydevops23/CI_CD/assets/126971054/9daff864-fbbb-4e99-bc50-233c6d943440)


Then you will be asked which plugins to install – choose **suggested plugins.**

![1_choose_suggested_plugin](https://github.com/ifydevops23/CI_CD/assets/126971054/386a2c0e-7ef8-4640-a42d-c65baa3fc160)


Once plugin installation is done – create an admin user and you will get your Jenkins server address. (Reload or open in a new tab incase of a freeze/unresponsive) <br>

![1_create_new_user](https://github.com/ifydevops23/CI_CD/assets/126971054/6b7c0db0-0ea3-4941-a386-d8b36c872f60)

![1_Jenkins_is ready](https://github.com/ifydevops23/CI_CD/assets/126971054/667075a2-6dd3-4339-abdb-b09b80da0fcb)

The installation is completed!<br>

**Incase proxy server is broken,implement this** - Reload or open in a new tab if neccessary.<br>

![0_jenkins_security](https://github.com/ifydevops23/CI_CD/assets/126971054/7a82b14a-e429-4344-838c-d90f33f45d15)

## STEP 2 – CONFIGURE JENKINS TO RETRIEVE SOURCE CODES FROM GITHUB USING WEBHOOKS.<br>

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). <br>
This job will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.<br>

- Enable webhooks in your GitHub repository settings

![2_add_webhook](https://github.com/ifydevops23/CI_CD/assets/126971054/86fa9c32-4016-4406-99cd-f5b90537185c)


![1_add_webhppk_payload_url](https://github.com/ifydevops23/CI_CD/assets/126971054/0eaa5653-8553-48db-baee-a106552d1cfa)

![2_successfully_created_webhook](https://github.com/ifydevops23/CI_CD/assets/126971054/44d062ff-f548-4667-a109-7c778e3c3aea)

- Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![8_create_job_freestyle](https://github.com/ifydevops23/CI_CD/assets/126971054/a5c5526c-f732-4f5b-9104-ee1c71a53d72)

- To connect your GitHub repository, you will need to provide its URL, you can copy it from the repository itself.

- In the configuration of your Jenkins freestyle project choose Git repository, and provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![3_add_credential_to_job](https://github.com/ifydevops23/CI_CD/assets/126971054/12fb7ab4-7545-4858-bc20-cadcfdf0e4a3)

- Save the configuration and let us try to run the build. 
For now, we can only do it manually.
Click the "Build Now" button, if you have configured everything correctly, the build will be successful and you will see it under #1

![1_configure_2](https://github.com/ifydevops23/CI_CD/assets/126971054/fa69e1f9-4823-414a-9188-b1c1a66517ee)

- You can open the build and check in "Console Output" if it has run successfully. If so – congratulations!<br>
You have just made your very first Jenkins build!But this build does not produce anything and it runs only when we trigger it manually.<br> 
Let us fix it.<br>

- Click "Configure" your job/project and add these two configurations.<br>
- Configure triggering the job from the GitHub webhook:<br>
![3_build_triggers](https://github.com/ifydevops23/CI_CD/assets/126971054/30ee6784-9cc1-4448-b437-6cfcefe49c12)

- Configure "Post-build Actions" to archive all the files – files resulting from a build are called "artifacts".<br>

![2_post_build_action](https://github.com/ifydevops23/CI_CD/assets/126971054/9be56b34-5184-46f7-a8da-e75ef9c545aa)

- Now, go ahead and make some changes in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.<br>
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on the Jenkins server.<br>

![6_Build2_archiving_artifacts](https://github.com/ifydevops23/CI_CD/assets/126971054/00b5c5c6-8ff8-4f87-ba33-eef07f03be14)

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and file transfer is initiated by GitHub).<br> There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.<br>
By default, the artifacts are stored on the Jenkins server locally<br>
`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

![8_archive_from_putty](https://github.com/ifydevops23/CI_CD/assets/126971054/e000fcd2-4049-44ce-a038-a2d15c71cf58)

## STEP 3 - CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.<br>
Jenkins is a highly extendable application and there are 1400+ plugins available. <br>

We will need a plugin that is called "Publish Over SSH".<br>

1. Install the "Publish Over SSH" plugin.
- On the main dashboard select "Manage Jenkins" and choose the "Manage Plugins" menu item.
- On the "Available" tab search for the "Publish Over SSH" plugin and install it.

![4_publish_over_ssh](https://github.com/ifydevops23/CI_CD/assets/126971054/bc5a5cab-8e62-4192-94dd-7708df7c92ca)

![8_installed_plugin](https://github.com/ifydevops23/CI_CD/assets/126971054/452906a1-3be0-42bf-9b76-6bb8ed2b6b08)


2. Configure the job/project to copy artifacts over to the NFS server.
- On the main dashboard select "Manage Jenkins" and choose the "Configure System" menu item.
- Scroll down to Publish over the SSH plugin configuration section and configure it to be able to connect to your NFS server:
![5_Publish_over_ssh](https://github.com/ifydevops23/CI_CD/assets/126971054/b83ce401-79f6-4b6d-b22d-e94693bbf137)

- Provide a private key (the content of .pem file that you use to connect to the NFS server via SSH/Putty)
![cat_nfs_key](https://github.com/ifydevops23/CI_CD/assets/126971054/4ec45a5b-87c2-406d-b6d7-db6fd6804cbb)
- Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since the NFS server is based on EC2 with RHEL 8)
- Remote directory – /mnt/apps since our Web Servers use it as a mounting point to retrieve files from the NFS server.
Test the configuration and make sure the connection returns Success. <br>
**Remember, that TCP port 22 on NFS server must be open to receive SSH connections.**
![5_success_from_nfs](https://github.com/ifydevops23/CI_CD/assets/126971054/83031d45-07da-4608-8dc6-2a3f976b159e)

- Save the configuration,open your Jenkins job/project configuration page and add another one "Post-build Action".

- Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use ** <br>

![8_send_build_over_ssh](https://github.com/ifydevops23/CI_CD/assets/126971054/4456f48b-0f18-4373-8e84-0e03aea4964e)

- Save this configuration and go ahead, and change something in README.MD file in your GitHub Tooling repository.
Webhook will trigger a new job and in the "Console Output" of the job you will find something like this: <br>

![6_Build3_transfer_via_ssh](https://github.com/ifydevops23/CI_CD/assets/126971054/5e83dd75-0b50-4ff3-b282-47420e5e7490)

```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```

![5_Console_output](https://github.com/ifydevops23/CI_CD/assets/126971054/b468321b-8daf-4ce1-89db-59688ee97d69)

To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file<br>
`cat /mnt/apps/README.md`<br>

![7_putty_response_from_nfs](https://github.com/ifydevops23/CI_CD/assets/126971054/52368aad-8f60-4571-91c2-141c21ffaf2c)

If you see the changes you had previously made in your GitHub – the job works as expected.<br>

Congratulations!You have just implemented your first Continous Integration solution using Jenkins CI. 

