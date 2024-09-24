# DevSecOps Project: Netflix Clone Deployment with CI/CD Pipeline

## Workflow Diagram of my Project -
![netflix-workflow](Blogjpg.jpg)

# Description 
Welcome to my DevSecOps project where I deploy a Netflix clone application using modern DevOps practices and tools. In this project, I leverage Jenkins for Continuous Integration and Continuous Deployment (CI/CD), Docker for containerization, and integrating different tools with each other, Kubernetes for orchestration, and monitoring tools like Grafana, Prometheus, and Node Exporter for observability.

## Project Overview
This project aims to demonstrate a complete DevSecOps workflow for deploying a Netflix clone application. The workflow includes:

- **Continuous Integration/Continuous Deployment (CI/CD)**: Using Jenkins, we automate the build, test, and deployment process of our Netflix clone application. Jenkins will pull code from our version control system, run tests, build Docker images, and deploy to Kubernetes.

- **Containerization with Docker**: We containerize our Netflix clone application using Docker. This enables consistency in deployment across different environments and simplifies dependencies.

- **Orchestration with Kubernetes:** Our application will be deployed and managed on a Kubernetes cluster. Kubernetes helps in scaling, load balancing, and maintaining high availability of our application.

- **Monitoring with Grafana and Prometheus**: We monitor the health and performance of our Jenkins CI/CD pipeline as well as our Kubernetes cluster using Grafana for visualization and Prometheus for data collection.

# Tools Used

- **Jenkins:** Automates the CI/CD pipeline and orchestrates the deployment process.

- **Docker**: Enables containerization of our Netflix clone application.

- **SonarQube**: continuous inspection of code quality and security vulnerabilities in software development projects.

- **Trivy**: It a vulnerability scanner for containers, used to detect security issues and vulnerabilities in container images and filesystems.

- **Kubernetes**: Manages and orchestrates the deployed containers in a scalable manner.

- **Grafana**: Visualizes and monitors Jenkins and Kubernetes metrics.

- **Prometheus**: Collects and stores metrics data from Jenkins and Kubernetes.

- **Node Exporter:** Exposes machine-level metrics for Prometheus.

- **Email Notification**: Using Jenkins Email Extension you will get all the notification what have you done in your cicd pipeline about Trivy and all.

# Getting Started-

## Prerequisites
Before setting up this project, ensure you have the following prerequisites:

- **AWS Account:** You will need an AWS account to provision EC2 instances. If you don't have an AWS account, you can [create one here](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26nc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgsFromTB_ap-southeast-2_aacbff57e379dd11&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&forceMobileApp=0&code_challenge=gz0r28YwtQey_P0ZDDgYsMbdjUHsI0LNhXn3s58m1nU&code_challenge_method=SHA-256).

    - Launch an EC2 instance of type t2.large with Ubuntu AMI, providing at least 40 GB of storage. This instance will be used for running Jenkins.
    - Launch another EC2 instance of type t3.medium for hosting Grafana, Prometheus, and Node Exporter for monitoring purposes.

- **SSH Access**: Connect to your AWS EC2 instances using SSH or any preferred method for configuration and setup.

---
Now install Jenkins-
```bash
#!/bin/bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

After running this script, use your `<EC2-instance-IP:8080>` and access it via the browser.

- after that install docker using these command-
   ```bash
   sudo apt install
   ```

   ```bash
   sudo apt update -y
   ```

   ```bash
   sudo apt install docker.io -y
   ```

- Docker group for permissions.

   ```bash
   sudo usermod -a -G docker $USER
   ```

- Switch primary group to Docker for current session.

   ```bash
   newgrp docker
   ```

- Granting universal read, write, and execute permissions to Docker socket.

    ```bash
    sudo chmod 777 /var/run/docker.sock
    ```

- now install sonarqube using this-
    ```bash
    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
    ```

- now install the trivy for scanner for containers, used to detect security issues and vulnerabilities in container images and filesystems.

  ```bash
  sudo apt-get install wget apt-transport-https gnupg lsb-release
  wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
  echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
  sudo apt-get update
  sudo apt-get install trivy -y
  ```

- now that's all now install the `Grafana`, `Prometheus`, and `Node Exporter` and Integrate via Jenkins.

- after all then Install plugins on Jenkins-
- Plugins Prerequisites in Jenkins-
    1. `Prometheus metrics`
    2. `Email Extension Templates`
    3. `Eclipse Temurin Installer`
    4. `SonarQube Scanner`
    5. `NodeJS`
    6. `OWASP Dependency-Check`
    7. `Docker`
    8. `Docker commons`
    9. `Docker Pipeline`
    10. `Docker API`
    11. `Docker-build-step`

- Now, in Jenkins credentials, add the credentials for `Email`, `SonarQube`, and `DockerHub`.

- Now run the Pipeline Script and wait at least 30 minutes because it takes 30 minutes to execute all dependencies and tasks.

 ```bash
   pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/sachin2815/DevSecOps-CICD-Pipeline-NetflixClone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
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
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build --build-arg TMDB_V3_API_KEY=ca86fe14eca3e76864bc17f59d319b92 -t netflix ."
                       sh "docker tag netflix sachin2815/netflix:latest "
                       sh "docker push sachin2815/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sachin2815/netflix:latest > trivyimage.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 sachin2815/netflix:latest'
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'sachin.singh.professional@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}

```

- According to your changes, update your Docker Hub email, Sonar token, and everything in the pipeline; do not simply copy the same as before.
- After completing this, check your Grafana to review your Jenkins aborted jobs and other details. Then access your `Netflix-EC2-public-IP with port 8081` to view the Netflix deployment. Remember to use your own TMDB API key in Docker build and push Stages in Pipeline.

<img src="https://www.animatedimages.org/data/media/562/animated-line-image-0184.gif" width="1920" />

# Conclusion
This project demonstrates the power of DevSecOps in deploying a modern application like a Netflix clone. By automating the build, test, and deployment processes and incorporating monitoring tools, we ensure a robust and scalable deployment pipeline. I hope you find this project useful and insightful!


<img src="https://www.animatedimages.org/data/media/562/animated-line-image-0184.gif" width="1920" />

---

***Now, from here, you can visit [my medium blog](https://medium.com/@sachin28), where you will find everything explained from scratch, with screenshots for every step, uploaded from the beginning to the end. So once visit on [DevSecOps-Netflix Blog for every steps](https://medium.com/@sachin28/deploying-a-netflix-clone-ci-cd-pipeline-with-jenkins-docker-kubernetes-monitoring-2f3ee4f758fe).***

---

***That’s all from my side. If you encounter any issues while working on this project, feel free to connect with me via-***

- [LinkedIN](https://www.linkedin.com/in/sa-chin/)
- [Discord](https://discord.com/channels/@me)
- [Mail Me](mailto:sachin.profess@gmail.com)

**Your insights are valuable and will help improve this project for everyone. Don't hesitate to connect!** ✨😊
