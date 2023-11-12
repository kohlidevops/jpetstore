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
    wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
    echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
    sudo apt update -y
    sudo apt install temurin-17-jdk -y
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

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/795aac40-b076-499a-b6a9-4543e94355d0)



