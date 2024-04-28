### End-to-End CI/CD Pipeline Implementation

![devops-diagram.png](https://github.com/itzchioma/terraform-configuration/blob/main/assets/devops-diagram.png)

### Introduction

+ Continuous integration and Continuous delivery (CI/CD) are crucial in software development modernization which helps to facilitate automated code integration and reliable application delivery.

+ Jenkins, known for its flexibility and extensive plugin options, is a leading tool for creating CI/CD pipelines.

+ We will cover everything from configuring Jenkins and integrating it with version control systems to orchestrating builds, tests, and deployments.

### We will be utilizing a variety of technologies and tools in this guide, including:

+ GitHub for version control
+ Maven for project management and builds
+ SonarQube for code quality analysis
+ Docker for containerization
+ Jenkins for Continuous Integration
+ ArgoCD and Helm for Kubernetes deployment management
+ Kubernetes for orchestrating containers

### Prerequisite
+ GitHUb Account
+ AWS Account (launch an Ec2 instance)

### Setting Up Jenkins
### Install Java:

#### Before you can run Jenkins, it’s essential to have Java installed on the server. Jenkins is compatible with both OpenJDK and Oracle Java, though it generally performs best with OpenJDK. 

#### Here’s how to install Java on the instance you’ve SSHed into:

<!-- Updating the ubuntu server (Prepping the server for configuration) -->

```bash
sudo apt update
sudo apt install openjdk-11-jdk
java -version
```

+ Install Jenkins:

+ Create a script file using ‘vim’ or any other editor of your choice.

```bash
vim install_jenkins.sh
```

+ Press i to ensure you are in insert mode and write your script.

```bash 
#!/bin/bash
# Download Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository to package manager sources
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Update package manager repositories
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y

```

+ Make the file executable using the following command:

```bash
chmod +x install_jenkins.sh 
```

+ Now that your script is executable, you can run it to install Jenkins:

```bash
./install_jenkins.sh
```

+ Adjust Firewall Settings:
Jenkins runs on port 8080 by default.

+ Accessing Jenkins UI: http://ip-address:8080

+ Unlock Jenkins by using the initial admin password found at: 

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Install Necessary Plugins:

+ Open Jenkins Dashboard: Log in to your Jenkins interface.
+ Navigate to “Manage Jenkins > Plugins”.
+ Install Plugins: Look for the “docker pipeline” and “sonarqube scanner” plugins install them and restart Jenkins if required.

### Compile Job
+ From the Jenkins main dashboard, click on “New Item”.
+ Name your pipeline and select ‘Pipeline’ as the type of project, then click ‘OK’.

### Configure Your Pipeline:
+ Click on the created job and scroll down to the “Pipeline” section in the configuration screen.
+ Choose “Pipeline script” or “Pipeline script from SCM”.

### Restart Jenkins:
+ Restart Jenkins to apply configuration changes or updates effectively.
+ To do so, navigate to the Jenkins “dashboard” and click on ‘Manage Jenkins’ in the sidebar.

### Set up Sonarqube Server
+ Installing SonarQube as a Docker container is a popular option that simplifies the setup process and makes it easier to manage and scale.
+ Prerequisites: Ensure Docker is installed on your server. If not, you can download and install Docker from the official Docker website.

### Docker Installation:

+ Create a script file using ‘vim’ or any other editor of your choice.

```bash
vim install_docker.sh
```

+ Press i to ensure you are in insert mode and write your script.

```bash 
#!/bin/bash

# Update package manager repositories
sudo apt-get update

# Install necessary dependencies
sudo apt-get install -y ca-certificates curl

# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package manager repositories
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
```
+ Press Esc to ensure you are in normal mode then type:wq and then press Enter.
+ Make the file executable using the following command:

```bash
chmod +x install_docker.sh
```
+ Now that your script is executable, you can run it to install Docker:

```bash
./install_docker.sh
```

### Install Sonarqube:

+ Pull the official SonarQube Docker image from Docker Hub:

```bash
docker pull sonarqube
```

+ Run SonarQube in a Docker container, using the following command.

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube
```

+ SonarQube operates on port 9000 by default. Therefore, to ensure seamless access to the SonarQube dashboard, it’s essential to configure your firewall to allow inbound traffic on this port.

+ The default login credentials for SonarQube are:

- Username: admin
- Password: admin

### Integrate with Jenkins:

+ Install the SonarQube Scanner for the Jenkins plugin.
+ Log in to your SonarQube dashboard.
+ Go to “My Account” > “Security” Click on “Generate Token”
+ Provide a name for the token and click “Generate”.
+ Copy the generated token.

### Add SonarQube Token as Credential in Jenkins:

+ In Jenkins, go to “Manage Jenkins” >“Credentials” > “System” > “Global credentials” (or navigate to your project’s credentials).
+ Click “Add Credentials”.
+ Choose “Secret text” as the kind of credential.
+ Paste the SonarQube authentication token into the “Secret” field.
+ Optionally, provide an ID and a description of the credential.
+ Click “Create” to save the credential.

### Configure Jenkins SonarQube Scanner:

+ In your Jenkins job configuration, find the section for SonarQube analysis or whatever you have named it.
+ Provide the SonarQube server URL (e.g.,http://<your_instance_ip>:9000, replacing <your_instance_ip> with your server's IP address).
+ Use the previously added SonarQube token as the authentication token.

### Credentials

+ Ensure that all required credentials are properly configured for your CI/CD pipeline.
+ This includes credentials for SonarQube authentication, Docker Hub access, and Git repository authentication.

### Jenkinsfile

+ A Jenkinsfile is a text file that defines the configuration of a Jenkins pipeline. It is written in Groovy, a scripting language for the Java platform.
+ The Jenkinsfile specifies the steps, stages, and actions that Jenkins should execute when running a pipeline job.

## Pipeline Stages:

+ Stage 1: Checkout the source code from Git.
+ Stage 2: Build the Java Application using Maven.
+ Stage 3: Run unit tests using JUnit and Mockito.
+ Stage 4: Run SonarQube analysis to check the code quality.
+ Stage 5: Package the application into a JAR file.
+ Stage 6: Deploy the application to a test environment using Helm.
+ Stage 7: Run User acceptance tests on the deployed application.
+ Stage 8: Promote the application to a production environment using Helm.

```bash 
pipeline {
  agent {
    docker {
      image 'maria/docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/itzchioma/jenkins-CICD.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd spring-boot-app && mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://ipaddress:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'cd spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "jenkins-CICD"
            GIT_USER_NAME = "itzchioma"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "itzchioma10@gmail.com"
                    git config user.name "itzchioma"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push @github.com/${GIT_USER_NAME}/${GIT_REPO_NAME">https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
```
+ Click on “Build Now” to trigger a build of your pipeline job.
+ Jenkins will fetch the Jenkinsfile from your repository and execute it as defined.
+ View the progress of your pipeline job on the Jenkins dashboard.
+ Click on the job to view detailed logs and status updates as each stage of the pipeline is executed.
+ If there are any issues during pipeline execution, review the Jenkinsfile and job configuration for errors.
+ Check the console output and logs for more information on any failures.

### SonarQube will contain the report of the pipeline execution.


### Set Up AgroCD

+ ArgoCD manages the continuous deployment segment of CI/CD pipelines, automating deployments to Kubernetes.
+ You can either have local deployment using Minikube or Cloud Deployment using Amazon EKS.

### Prerequisites:

+ Ensure VirtualBox or Hyper-V is installed on your Windows machine for virtualization, as required by Minikube.

### Install Minikube:

+ Download and install Minikube following the instructions specific to your OS from the Minikube official documentation.
+ Start your local Kubernetes cluster.

```bash
minikube start
```

Install Kubectl:
+ Download the latest version of kubectl from the official Kubernetes release page.
+ Add kubectl to your PATH to run it from anywhere in your command prompt.

### Install ArgoCD Operator
## You can install Argo CD on Kubernetes using the Argo CD Operator which automates the deployment and management of Argo CD instances.

+ Go to the official Operator Hub page at OperatorHub.io.
+ Use the search bar on the Operator Hub website to search for “Argo CD” and click “Install”.
+ Run the Commands the following commands:

```bash 

#Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.

$ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.27.0/install.sh | bash -s v0.27.0
```

```bash 
#Install the AgroCD Operator
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```

+ This Operator will be installed in the “operators” namespace and will be usable from all namespaces in the cluster.

```bash 
# watch your operator come up 
$ kubectl get csv -n operators
```
### Set Up AgroCD Controller

+ Navigate to OperatorHub.io.
+ In the “Argo CD” Operator scroll down to “Operator Documentation”.
+ Click on “Usage” and then “Basics”.
+ Copy the YAML configuration provided. This YAML is used to deploy Argo CD in your Kubernetes cluster.
+ Create a new file named vim argocd-basic.yml with the following content to define your Argo CD instance:

```bash 
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```

+ Apply the Configuration.

```bash
kubectl apply -f argocd-basic.yml
```

### Set Up AgroCD UI

+ To access the Argo CD server UI via the browser, you need to change the service type from ‘ClusterIP’ to ‘NodePort’.

```bash
kubectl get svc
```

+ Minikube can generate a URL that provides direct access to the Argo CD server through a browser.

```bash 
minikube service argocd-server --url
```

+ Copy the URL displayed from the previous command into your browser to access the Argo CD UI.
+ The default username is ‘admin’. To get the admin password, you need to extract it from Kubernetes secrets:

```bash
kubectl get secret
```

+ Edit the “example-argocd-cluster” secret and copy the admin password.

```bash
kubectl edit secret example-argocd-cluster
```

+ K8s secrets are base 64 encrypted so to decode it use this command.

```bash
echo <encoded password here>= | base64 -d
```

+ Use the username ‘admin’ and the password retrieved in the previous step to log into the Argo CD UI.

### Deployment with Agro CD

+ In the Argo CD UI, click on “Create Application”
+ Fill in the required information for the application:

## Application Name: Enter a descriptive name for your application.
## Project Name: Specify the project to which the application belongs.
## Sync: Choose “Automatic” for automatic synchronization.
## Repository URL: Enter the URL of your Git repository containing the application code.
## Path: Specify the path to the deployment files within the repository.
## Destination: Enter the URL of your Kubernetes cluster (e.g., https://kubernetes.default.svc).
## Namespace: Specify the Kubernetes namespace where the application will be deployed.

+ After providing all the necessary information, click on “Create”.
+ Argo CD will automatically create the application on your Kubernetes cluster based on the provided configuration.









