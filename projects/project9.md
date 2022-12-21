# **CONTINUOUS INTEGRATION PIPELINE FOR TOOLING WEBSITE**
Enhance the architecture prepared in LOAD BALANCER SOLUTION WITH APACHE PROJECT by adding a Jenkins server, configure a job to automatically deploy source code changes from Git to NFS server.

# **Step 1 - Preparing prerequisites** 
In order to complete this project, an AWS account, Jenkins Server(based on Ubuntu 20.04), RHEL8 Web Servers, one MySQL DB Server (based on Ubuntu 20.04) and one RHEL8 NFS server is required.  
<br>

Creation of a new AWS account gives access to the free tier plan which allows to spin up a new EC2 instance (an instance of a virtual server) for free in only a matter of a few clicks.  
<br>

You can watch the videos below to learn how to Provision a server and connect to it.
- [AWS account setup and Provisioning an Ubuntu Server](https://www.youtube.com/watch?v=xxKuB9kJoYM&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=6) 
- [Connecting to your EC2 Instance](https://www.youtube.com/watch?v=TxT6PNJts-s&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=7)    
<br>

# **Step 2 - Install Jenkins server** 
Create an Ubuntu Server 20.04 EC2 instance for Jenkins, so your EC2 list will look like this:

![all instances running](../screenshots/project9/instances_running.jpg)
*All instances running*  
<br>

Install JDK (since Jenkins is a Java-based application)
>`sudo apt update`   
>`sudo apt install default-jdk-headless`   

![install java jdk](../screenshots/project9/install_jdk.jpg)
*Install java jdk*  
<br>

Install Jenkins
>`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`   
>`sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'`   
>`sudo apt update`   
>`sudo apt-get install jenkins`   

![install jenkins](../screenshots/project9/install_jenkins.jpg)
*Install jenkins*  
<br>

Make sure Jenkins is up and running   
>`sudo systemctl status jenkins`   

![jenkins running](../screenshots/project9/jenkins_running.jpg)
*Jenkins running*  
<br>

By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group   
![open port 8080](../screenshots/project9/open_port_8080.jpg)
*Open port 8080*  
<br>

Perform initial Jenkins setup.   

From your browser access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`   

Providing default admin password obtained from `/var/lib/jenkins/secrets/initialAdminPassword`      

![default admin password](../screenshots/project9/default_password.jpg)
*Default admin password*  
<br>

Then you will be asked which plugings to install – choose suggested plugins.   

![suggested plugins](../screenshots/project9/plugins.jpg)
*Suggested plugins*  
<br>

Once plugins installation is done – create an admin user and you will get your Jenkins server address.   

The installation is completed!   

![jenkins ready](../screenshots/project9/jenkins_ready.jpg)
*Jenkins ready*  
<br>

# **Step 3 - Configure Jenkins to retrieve source codes from GitHub using Webhooks**
In this part, we will configure a simple Jenkins job/project. This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.   
<br> 

Enable webhooks in your GitHub repository settings   

![enable webhooks](../screenshots/project9/enable_webhooks.jpg)
*Enable webhooks*  
<br>

Go to Jenkins web console, click "New Item" and create a "Freestyle project"   

![create freestyle project](../screenshots/project9/created_freestyle.jpg)
*Create freestyle project*  
<br>

In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.   

![configure freestyle project](../screenshots/project9/configure_freestyle_project.jpg)
*Configure freestyle project*  
<br>

Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1   

![build run](../screenshots/project9/build_run.jpg)
*Build run*  
<br>

You can open the build and check in "Console Output" if it has run successfully.   

If so – congratulations! You have just made your very first Jenkins build!   

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.   
<br>

Click "Configure" your job/project and add these two configurations   

Configure triggering the job from GitHub webhook:   
![triggering the job](../screenshots/project9/triggering_job.jpg)
*Triggering the job*  
<br>

Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".   
![post build actions](../screenshots/project9/post_build_actions.jpg)
*Post build actions*  
<br>

Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.   

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.   

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.    

![successful build trigger](../screenshots/project9/build_trigger.jpg)
*Successful build trigger*  
<br>


By default, the artifacts are stored on Jenkins server locally   

>ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/

# **Step 4 - Configure Jenkins to copy files to NFS server via SSH**   
The artifacts are saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.   

Install "Publish Over SSH" plugin" from the Manage Jenkins and select Manage Plugins under System Configuration.   

![install plugin](../screenshots/project9/install_plugin.jpg)
*Install plugin*  
<br>

Click on manage jenkins and then configure system. Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server.

![configure publish over ssh plugin](../screenshots/project9/configure_plugin.jpg)
*Configure publish over ssh plugin*  
<br>

Save the configuration, open your Jenkins job/project configuration page and add the "send build artifacts over ssh" post-build action.

Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories - so we use "**".

![configure to send all files](../screenshots/project9/configure_to_send_all_files.jpg)
*Configure to send all files*  
<br>

Make changes in the README.MD file in the git account and confirm if it will sync along with jenkins and show in the NFS server.

![change readme file](../screenshots/project9/change_readme_file.jpg)
*Change readme file*  
<br>

Below is the output from the NFS server, checking the README.MD file

![confirming nfs server sync](../screenshots/project9/confirming_sync.jpg)
*Confirming nfs server sync*  
<br>





















