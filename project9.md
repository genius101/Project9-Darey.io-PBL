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

