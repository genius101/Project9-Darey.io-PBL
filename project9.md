<h1> In this project, we will configure a Continous Integration Pipeline For our Tooling Website</h1>

<p>In this project we would enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.</p>

<p>Here is how your updated architecture will look like upon competion of this project:</p>

![add_jenkins](https://user-images.githubusercontent.com/10243139/128801351-03cc35e5-6929-4c6d-8512-74cdd144ff3f.png)

<h2>Step 1 – Install Jenkins server</h2>

<p>Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"</p>

<p>Install JDK (since Jenkins is a Java-based application)</p>

    sudo apt update
    sudo apt install default-jdk-headless
    
<p>Install Jenkins</p>

    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
        /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt-get install jenkins
    
<p>Make sure Jenkins is up and running</p>

    sudo systemctl status jenkins
    
<p>By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group</p>

![1 e](https://user-images.githubusercontent.com/10243139/128801943-287b6d37-ecd9-401c-871b-fae097f1f3c6.png)

<p>Perform initial Jenkins setup.</p>

<p>From your browser access http://Jenkins-Server-Public-IP-Address-or-Public-DNS-Name:8080</p>

<p>You will be prompted to provide a default admin password<p>

<p>Retrieve it from your server:</p>

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

![1 f](https://user-images.githubusercontent.com/10243139/128802171-54d19c2c-d956-4e5c-9adb-793369eadc99.jpg)

<p>Then you will be asked which plugings to install – choose suggested plugins.</p>

![1 g](https://user-images.githubusercontent.com/10243139/128802308-9b7685fd-7160-46ec-9d40-85aade2a4481.png)

<p>Once plugins installation is done – create an admin user and you will get your Jenkins server address.</p>

<p>The installation is completed!</p>

![1 g2](https://user-images.githubusercontent.com/10243139/128802354-e3f20d04-1c2e-4c28-ad12-95162dd4a745.jpg)


<h2>Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks</h2>


<p>Enable webhooks in your GitHub repository settings</p>

<p>Below is a gif on how to enable Webhooks:

![webhook_github](https://user-images.githubusercontent.com/10243139/128802580-b3214614-b684-4760-bd84-618ea326a867.gif)
    
<p>Here is my own webhook enable settings:</p>

![2 a1](https://user-images.githubusercontent.com/10243139/128802743-38947743-5e13-472c-b2bd-90641297a8e4.jpg)

<p>Go to Jenkins web console, click "New Item" and create a "Freestyle project" named tooling_github<p/>

<p>Connect your GitHub repository, you will need to provide its URL</p>

![2 c](https://user-images.githubusercontent.com/10243139/128802800-2aa6184c-1694-4dfc-ae98-d8ec26004c14.jpg)

<p>In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository</p>

![2 d](https://user-images.githubusercontent.com/10243139/128802889-06fafa15-1ddf-4862-bd8b-7d98c96e3c26.jpg)

<p>Save the configuration and let us try to run the build. Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under Build History</p>

![2 e](https://user-images.githubusercontent.com/10243139/128802957-b9dce2f0-1911-4efa-be36-6fe3e6af3370.jpg)

<p>But this build does not produce anything and it runs only when we trigger it manually.</p>

<p>To Fix it, Click "Configure" your job/project and add these two configurations:</p>

<p>Configure triggering the job from GitHub webhook:</p>

![2 f1](https://user-images.githubusercontent.com/10243139/128803144-5a2ce395-ae24-48e1-b76c-959887a398e6.png)

<p>Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".</p>

![2 f2](https://user-images.githubusercontent.com/10243139/128803186-2f1dc88a-f66b-4194-b144-de1a39d5b0ca.jpg)

<p>Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch via Visual Studio Code</p>

<p>You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.</p>

![2 h1](https://user-images.githubusercontent.com/10243139/128803513-ca33f0f0-6460-46b9-8adc-88aaf3296dd1.jpg)

<p>You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub).</p>

![2 h2](https://user-images.githubusercontent.com/10243139/128803576-21ffc310-e21b-474a-adf7-6450b38d2bc0.jpg)

<p>By default, the artifacts are stored on Jenkins server locally</p>

    ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/

![2 i](https://user-images.githubusercontent.com/10243139/128803705-6a3007f2-1654-4f24-8e83-5b371182d8e2.jpg)


<h3>Now, Let’s Configure Jenkins to copy files to NFS server via SSH</h3>


<h2>Step 3 – Configure Jenkins to copy files to NFS server via SSH</h2>

<p>Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.</p>

<p>Install "Publish Over SSH" plugin.</p>
	
<p>On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.</p>

<p>On "Available" tab search for "Publish Over SSH" plugin and install it(Install without Restart)</p>

![3 a](https://user-images.githubusercontent.com/10243139/128804014-a4d767a7-bae1-411a-bcd6-c0b08d26edf1.jpg)

<p>Configure the job/project to copy artifacts over to NFS server.</p>

<p>On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.</p>

<p>Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:</p>

    1. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
    2. Arbitrary name
    3. Hostname – can be private IP address of your NFS server
    4. Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
    5. Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
    
![3 b](https://user-images.githubusercontent.com/10243139/128804242-258cad33-5246-41fc-8503-c6c981af71ca.jpg)

<p>Test the configuration and make sure the connection returns Success.</p>

![3 b](https://user-images.githubusercontent.com/10243139/128804270-649daefd-b37f-43f9-b1ba-3f5819f8d639.png)

<p>Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"</p>

![send_build](https://user-images.githubusercontent.com/10243139/128804437-dda1b37e-7b41-40e9-8a7d-9fc04762fd4f.png)

<p>Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.</p>

![3 d](https://user-images.githubusercontent.com/10243139/128804504-e71983ab-9c0c-44b2-b20a-b1da99f29211.png)

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

<p>Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:</p>

![3 e](https://user-images.githubusercontent.com/10243139/128804569-27890161-0dbd-4fa0-9943-67df3deb3f11.jpg)

<p>To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check apps directory if the content has been updated</p>

![3 f](https://user-images.githubusercontent.com/10243139/128804610-870aff6d-8e98-4449-8480-d3e7a76ad5c8.jpg)

<h2>Congratulations!</h2>

<h3>You have just implemented your first Continous Integration solution using Jenkins CI</h3>

