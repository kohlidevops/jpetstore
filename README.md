# To deploy Java based application on Kubernetes Cluster using CICD 

I'm going to deploy my java based application in Docker Container and the K8S cluster. I have used below repository to deploying application.

    https://github.com/kohlidevops/jpetstore.git

## Step -1: Setup Jenkins

Launch new EC2 t2.large instance with Ubuntu-22 Image.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/d203a4bd-bb8c-46ed-82cf-08fcab180923)

### Install Jenkins

SSH to Jenkins instance and run below commands to install Jenkins
        
        sudo apt update -y
        sudo apt upgrade -y
        wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo apt-key add -
        echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
        sudo apt update
        sudo apt install temurin-17-jdk
        /usr/bin/java --version
        curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
        sudo apt-get update -y
        sudo apt-get install jenkins -y
        sudo systemctl start jenkins
        sudo systemctl status jenkins

After installation of Jenkins, I will create Inbound Port 8080, since Jenkins works on Port 8080.

But for my case, we are running Jenkins on another port. Because my application has to be use 8080 port. So, I'm going to change the port to 8090 using the below commands.

        sudo systemctl stop jenkins
        sudo systemctl status jenkins
        cd /etc/default
        sudo vi jenkins   
            <change port HTTP_PORT=8090 and save and exit>
        cd /lib/systemd/system
        sudo vi jenkins.service  
            <change Environments="Jenkins_port=8090" save and exit>
        sudo systemctl daemon-reload
        sudo systemctl restart jenkins
        sudo systemctl status jenkins

Now access the Jenkins webui using IP with port - 8090 and login the console then install suggested plugins.

### Install Docker in Jenkins instance

To install a docker and configure using below commands.

        sudo apt-get update 
        sudo apt-get install docker.io -y 
        sudo usermod -aG docker $USER 
        newgrp docker 
        sudo chmod 777 /var/run/docker.sock

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/795aac40-b076-499a-b6a9-4543e94355d0)

### To launch Sonarqube docker container in Jenkins instance

        docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Now, I can able to access the sonarqube docker container. Remember! default username is admin and password is admin. Then I have to reset the admin password.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/b5f152b5-35f7-4f33-8ac8-17b0520bbd20)

### Install Trivy in Jenkins instance

        sudo apt-get install wget apt-transport-https gnupg lsb-release -y
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
        echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy -y

## Step -2: Install Plugins

### Install Eclipse and Sonarqube scanner plugins in Jenkins

To install below plugins and restart the Jenkins.

        Eclipse Temurin Installer
        SonarQube Scanner

### Configure Java and Maven in Jenkins

To configure Java and Maven in Jenkins using Global tool configuration. Jenkins -> Manage Jenkins -> Tools

Add JDK

![0 GcyJCjumYC7TeNis](https://github.com/kohlidevops/jpetstore/assets/100069489/61a6e830-17bd-4f78-94d5-3195540c2e32)

Add Maven

![0 ko5xAW5n2MXQsgZc](https://github.com/kohlidevops/jpetstore/assets/100069489/ffc84fa3-0230-40c7-aeeb-d7324d05fb29)

