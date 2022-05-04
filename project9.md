# Documentation of Project 9: Continous Integration Pipeline For Tooling Website

## Step 1: Install Jenkins Server

- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"


- Install JDK (since Jenkins is a Java-based application)

```sh
sudo apt update
sudo apt install default-jdk-headless
```
- Install Jenkins
```sh
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```

- Make sure Jenkins is up and running: `sudo systemctl status jenkins`
![jenkins-running](./images/jenkins-running.PNG)

- By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
![jenkins-sg](./images/jenkins-sg.PNG)

- Perform initial Jenkins setup.
From your browser access: http://< Jenkins-Server-Public-IP-Address-or-Public-DNS-Name >:8080
![jenkins](./images/jenkins.PNG)

- You will be prompted to provide a default admin password. Retrieve it from your server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

- Then you will be asked which plugins to install – choose suggested plugins.
![jenkins-2](./images/jenkins-2.PNG)

- Once plugins installation is done – create an admin user and you will get your Jenkins server address.
![jenkins-3](./images/jenkins-3.PNG)

- The installation is completed!
![jenkins-ready](./images/jenkins-ready.PNG)
![jenkins-welcome](./images/jenkins-welcome.PNG)

# Step 2 – Configure Jenkins to Retrieve Source Codes from GitHub Using Webhooks

- Enable webhooks in your GitHub repository settings
![webhook](./images/webhook.PNG)

- Go to Jenkins web console, click "New Item" and create a "Freestyle project"


- To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

- Save the configuration and try to run the build. Click "Build Now" button, if you have configured everything correctly, the build will be successful and you will see it under #1. You can open the build and check in "Console Output" if it has run successfully.
![console](./images/console.PNG)
But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

- Click "Configure" your job/project and add these two configurations

a. Configure triggering the job from GitHub webhook.

b. Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

- Now, make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

By default, the artifacts are stored on Jenkins server locally:`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`


## Step 3: Configure Jenkins to Copy Files to NFS Server via SSH

- Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

- On "Available" tab search for "Publish Over SSH" plugin and install it
![install-plugins](./images/install-plugins.PNG)

- Configure the job/project to copy artifacts over to NFS server. On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server
![ssh-config](./images/ssh-config.PNG)

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
![private-key](./images/private-key.PNG)

- Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

- Save the configuration, open your Jenkins job/project configuration page and add another "Post-build Action". Configure it to send all files produced by the build into previously defined remote directory. In our case we want to copy all files and directories – so we use **.
![send-artifacts](./images/send-artifacts.PNG)

- Save this configuration and change something in README.MD file in your GitHub Tooling repository.

- To make sure that the files in /home/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file
![proof](./images/proof.PNG)



