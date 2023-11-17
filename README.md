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

### Install OWASP Dependency check

Jenkins console → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/97c2a9f1-a6de-415f-9823-187be1778e35)

In order to configure OWASP in Jenkins Tools

Jenkins console -> Manage Jenkins -> Tools

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/b73f9964-53ff-4b84-b72e-d8f1834b19f7)

Apply and save it.

### Add OWASP stage in Pipeline

Jenkins console -> select your Job -> Navigate to Pipeline and add below stages

        stage ('Build WAR file'){
                    steps{
                        sh 'mvn -N io.takari:maven:wrapper'
                        sh 'mvn clean install -DskipTests=true'
                    }
                }
                stage("OWASP Dependency Check"){
                    steps{
                        dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    }
                }

Apply and save it - Then start the build to see the result.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/c819e655-2122-4dfd-a482-681f6aced00d)


## Step -5: Docker Image Build and Push Stage

### Install Docker plugins

Jenkins console → Manage Plugins → Available plugins → Search for Docker and install these plugins.

        Docker
        Docker Commons
        Docker Pipeline
        Docker API
        docker-build-step

Click install without restart.

### Configure docker in Jenkins Tools

Jenkins console -> Manage Jenkins -> Tools -> Docker Installations -> Add Docker

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/338c7ac8-3091-4c0e-b5ba-592d8f7e2c74)

Apply and save.

### Add docker credentials in Jenkins Global credentials

Jenkins console -> Manage Jenkins -> Credentials -> System -> Global credentials -> Add -> User nme and password

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/6e77441e-c51f-4ccb-81f1-3e11f0bbef5b)

Then create.

### Add Docker build & push, Trivy scanning and Deploy docker container on Jenkins machine

Add below stages in existing pipeline script and start the build to see the result.

        stage ('Build and push to docker hub'){
                    steps{
                        script{
                            withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                                sh "docker build -t petshop ."
                                sh "docker tag petshop latchudevops/petshop:latest"
                                sh "docker push latchudevops/petshop:latest"
                           }
                        }
                    }
                }
                stage("TRIVY"){
                    steps{
                        sh "trivy image latchudevops/petshop:latest > trivy.txt"
                    }
                }
                stage ('Deploy to container'){
                    steps{
                        sh 'docker run -d --name pet1 -p 8080:8080 latchudevops/petshop:latest'
                    }
                }

This build stage will build the docker image using below dockerfile.

        https://github.com/kohlidevops/jpetstore/blob/main/Dockerfile

After the build image, the image should push to Docker repository.

Then this image will scanned by Trivy before deploy on docker container in Jenkins machine.

The build has been succedded.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/ed2c6746-f601-448e-8b49-d7605f722ab4)

I can able to see my docker container in Jenkins machine.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/b36052b4-13e6-44de-8a9f-6044c7b90915)

If i hit my URL with Port 8080 - Because myapp listening on Port 8080.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/adaa8de2-e8e8-4447-982e-1eb0ce8f00b2)

I can able to see my Images in Docker hub repository.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/54b31cd6-ad71-4030-803b-c8fb8213a0c5)

## Step -6: To setup Kubernetes Cluster

### Install kubectl on Jenkins

SSH to Jenkins machine and install below things to make available kubectl.

        sudo apt update
        sudo apt install curl
        curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
        sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
        kubectl version --client

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/ddcd1089-90e5-4998-9a52-ce7d869a02d0)

#### Launch Master and Worker node for Kubernetes

To launch two t3.medium ubuntu-20 machines for Kubernetes Master and Worker node.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/806dd4f4-dac3-4fa9-bd88-bb9c9527100e)

### Install docker, kubelet, kubeadm and kubectl

#### To install below commands on both kubernetes master and worker node.

        sudo apt-get update 
        sudo apt-get install -y docker.io
        sudo usermod –aG docker ubuntu
        newgrp docker
        sudo chmod 777 /var/run/docker.sock
        sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
        sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
        deb https://apt.kubernetes.io/ kubernetes-xenial main
        EOF
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo snap install kube-apiserver

#### To install below commands in kubernetes master node
        
        sudo kubeadm init --pod-network-cidr=10.244.0.0/16
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#### To install below commands in kubernetes worker node

        sudo kubeadm join 172.31.39.217:6443 --token p9nqcl.t37afz1pxvz2ubls \
                --discovery-token-ca-cert-hash sha256:12d30c9d1c32738701c7247240502524188bc4ec402bc07d7dc4b77b0dbea507

#### To copy config file in kubernetes master node to configure in Jenkins console

cd .kube
cat config

##### To copy and paste this content in local server and this file called as Secret File.txt. I will use this file in jenkins later.

### Step -7: To conigure secret file in Jenkins console

To configure the Secret File.txt (which is created in last step) in Jenkins Global credentials.

Jenkins -> Manage Jenkins -> Credentials -> System -> Global credentials -> New credentials -> Secret file -> upload the text file (Secret File.txt)

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/381b59b4-8113-46f3-beec-ac7f78092fe7)

Thats it! save the credentials.

#### To install kubernetes plugins

To install kubernetes plugins in Jenkins console.

Jenkins -> Manage Jenkins -> Plugins -> Available -> Select and install below plugins without restart

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/a89329cf-bef7-4a2d-99c4-c6090dd08193)

### Step -8: To configure Mail server in Jenkins

To configure mail server in Jenkins to receive notification when build has performed actions such as passed, failed and so on.

#### To install Email plugins in Jenkins console

Jenkins -> Manage plugins -> Available -> install below plugin.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/f41f554b-24f0-4040-891d-d3fd43730973)

#### Tuning your Gmail to receive mails

Go to your Gmail account and click on your profile. Then click on Manage Your Google Account -> click on the security tab on the left side panel you will get this below page.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/daf395a0-ba55-4e85-aef5-c883f18381e4)

2-step verification should be enabled. Search for the app in the search bar and you will get a app passwords like the below image.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/dcee4974-3b73-4618-a64f-272574111af4)

Then create a App name as Jenkins or any meaning ful name and create a password -> Then note it for later use.

#### To configure Email notification in Jenkins console

Jenkins -> Manage Jenkins -> System

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/5cf3318c-65ee-4e81-9636-9636688595ab)

Note: Password shoudl be generated password for app in last step.

Then apply and save.

#### To configure gmail credentials in Jenkins credentials manager

Jenkins -> manage jenkins -> credentials -> system -> global credentials -> add user name and password

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/9447863c-c116-4095-9871-c2454c3a5c11)

Note: Password shoudl be generated password for app in last step.

Then create the credentials

#### To verify the Email configuration in Jenkins

Jenkins -> Manage jenkins -> System -> under Extended Email notification

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/7b37ae1a-f0bf-435a-a2b1-03aa02e362c4)

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/19c50d30-61d7-48a9-aa6c-0567dc2b00c3)

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/9d4c4326-3308-445f-b275-897db2d4b514)

Then Apply and save. You can Test out too before start build.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/72b4bc14-e152-437e-a3d2-13d5b5b3a728)

#### To add a kubenetes deployment stage

To add below stage in your pipeline -> This stage will use the global credentials to deploy the app on kubernetes worker node.

        stage('K8 deployment stage'){
                    steps{
                        script{
                            withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yaml'
                            }
                        }
                    }
                }

Apply and save.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/ab1cbc9f-b43b-43c4-afc5-0c089c46f687)

#### To add a Mail notification in Pipeline

###### This mail post block should be after stages

        post {
             always {
                emailext attachLog: true,
                    subject: "'${currentBuild.result}'",
                    body: "Project: ${env.JOB_NAME}<br/>" +
                        "Build Number: ${env.BUILD_NUMBER}<br/>" +
                        "URL: ${env.BUILD_URL}<br/>",
                    to: 'latchu.devops8@gmail.com',
                    attachmentsPattern: 'trivy.txt'
                }
            }

Apply and save.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/710cdfad-94b4-4815-b1f9-b107e02bc48c)

Now start the build to see the results of all the stages.

My build has been succeded as i expect.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/abda8d27-b5e8-47e7-a40d-21000bab0ceb)

If i'm going to check with my kubernetes master with below command after the build.

        kubectl get all

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/6b61743f-dbf5-4ac9-b65f-c97744099d77)

I can able to see the my kubernetes worker node is running with my docker app.

Now try to access the kubernetes worker node public ip and port number. Here we go!

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/55c4f9c8-3460-4b25-a80e-09920ba6f23d)

I can check with my email to ensure the receiving email reports.

![image](https://github.com/kohlidevops/jpetstore/assets/100069489/8aa12242-93b2-4fc0-af84-7e1ee440c479)

That's it! 

#### Please terminate all the resource once test out.
