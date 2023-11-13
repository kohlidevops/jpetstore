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

## Step -3: Create a Jenkins job

To create a jenkins job with pipeline

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/65af51a4-39d7-4bee-ac56-732820ef30b1)

Im going to keep the maximum number of build as 4

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/47878f77-cbd9-4dee-9fb2-f4404e7f0ac0)

Navigate to Pipeline and select Pipeline script and paste the below script then check whether its working or not

        pipeline{
            agent any
            tools {
                jdk 'jdk17'
                maven 'maven3'
            }
            stages{
                stage ('Clean Workspace'){
                    steps{
                        cleanWs()
                    }
                }
                stage ('Git Checkout') {
                    steps {
                        git branch: 'main', url: 'https://github.com/kohlidevops/jpetstore.git'
                    }
                }
                stage ('Maven Compile') {
                    steps {
                        sh 'mvn clean compile'
                    }
                }
                stage ('Maven Test') {
                    steps {
                        sh 'mvn test'
                    }
                }
           }
        }

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/edb91b76-c95c-4f20-af19-d27de7a41305)

Now start the build to see the result. Perfect My build has been succeeded.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/00819016-efd2-47d5-954a-6e709fcad992)

## Step -4: Configure Sonarqube

I have launched Sonarqube application using docker container in Jenkins server. You can access the Sonarqube using the Jenkins IP address with Port 9000.

By default, user name and password is "admin" for sonarqube. Then we have to reset the password.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/54f0baf9-ccc1-401e-9b95-1a68415c1f35)

### Generate a Token in Sonarqube

To access the Sonarqube application from Jenkins, we have to create Token in Sonarqube. You can navigate using below steps.

Sonarqube Application -> Login -> Administration → Security → Users → Click on Tokens and Update Token → Meaningful name → and click on Generate Token

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/32c1388d-f1fe-4c0d-8992-6f1de31f0385)

This token will shown for one time. So save it locally for later use.

### Configure Sonarqube token in Jenkins

Navigate to Jenkins console and do below steps to save Sonarqube token as securely.

Jenkins console -> Manage Jenkins -> Credentials

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/1e759502-a148-4fe7-b10e-c923c6a0b48c)

Add Credentials -> Secret Text -> Paste the Sonarqube token in Secret label -> provide meaningful name and save it.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/a09e06cc-6fd1-4020-be4f-d8fb6180124e)

### Configure Sonarqube server in Jenkins

Navigate to Jenkins console -> Manage Jenkins -> System -> Sonarqube servers -> Sonarqube installation -> Add

Provide a meaningful name -> Server URL (Sonarqube URL with port) -> Server authentication token - Select the token which is created just before in Jenkins credentials.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/f79e2d40-9f53-4d29-90c5-18a071a7a04b)

Apply and save.

### Install Sonarscanner in Jenkins

To install a Sonarscanner in Jenkins console using Global Tool.

Jenkins -> Manage Jenkins -> Tools

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/f94717a4-b011-4e19-a157-db6201738b99)

Select -> Sonarqube scanner installation -> Add

Provide a meaningful name and install sonarscanner from Maven central.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/22271c9e-af4a-42db-a8cd-7cca1ca28d3f)

Apply and save it.

### Configure Webhooks in Sonarqube application

Login to Sonarqube application -> Administration -> Configuration -> Webhooks -> Create a webhook

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/3e321b46-714c-45c2-a77e-9dac43af1d29)

Provide a meaningful name and URL should be "Jenkins-URL:Port/sonarqube-webhook/ and create it.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/9cfe3c40-1528-4d41-a661-4aa157cf428f)

### Add Sonarqube stage in Jenkins pipeline

To add a Sonarqube stage in Jenkins pipeline using below code

#### under tools section add this environment
        environment {
                SCANNER_HOME=tool 'sonar-scanner'
            }
#### in stages add this
        stage("Sonarqube Analysis "){
                    steps{
                        withSonarQubeEnv('sonar-server') {
                            sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petshop \
                            -Dsonar.java.binaries=. \
                            -Dsonar.projectKey=Petshop '''
                        }
                    }
                }
                stage("quality gate"){
                    steps {
                        script {
                          waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                        }
                   }
                }

tool 'sonar-scanner' -> I have configured with this name in Jenkins Global tool configuration

Environamet 'sonar-server' -> I have installed sonar-scanner from Maven in Jenkins System with this same name

Token 'Sonar-token' -> I have created with this Id in Jenkins credentials for Quality status check

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/afacf9d5-535d-4053-bd02-b72ab7ed0967)

Now, update this code in pipeline and start the build again.

This build too successfully completed without fail.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/376fe8c8-a8b2-47f0-823e-9accd7c4232c)

If you want to check with sonarqube application for code analysis, Then please logon to the sonarqube application check the code status.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/4badc36c-cc2c-4d92-8191-f564665b9ee1)





