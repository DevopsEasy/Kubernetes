## Devops CI/CD Pipeline

![Preview](./pipeline.png)

## Source Code: GIT Necessary Sotwares
* GIT, Java, Maven, Docker, K8s, Jenkins
* Cloud: AWS
* Launch an ec2 instance in aws for Jenkins

## Jenkins Installation
* For jenkins to work java is mandatory
* Java installation

```
sudo apt update && sudo apt install openjdk-8-jdk -y
```
* Jenkins Installation [Refer Here](https://www.jenkins.io/download/)

```
wget https://get.jenkins.io/war-stable/2.303.3/jenkins.war
nohup java -jar jenkins.war &
# enter
```
* To Build the Java related projects maven is mandatory

```
sudo apt update && sudo apt install maven -y
```
* After building the code we need to convert it into a Docker Image
* Docker Installation [Refer Here](https://get.docker.com/)

```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
# exit & relogin
docker version
```
* First we need to choose java related opensource project
* I will be using Game Of Life to implement CI/CD [Refer Here](https://github.com/wakaleo/game-of-life)
* After generating artifact[war file] we need to convert into Docker Image
* We can convert using Dockerfile

```
FROM tomcat:8-jdk8
MAINTAINER DEVOPSEASY
ADD ./gameoflife-web/target/gameoflife.war /usr/local/tomcat/webapps/
EXPOSE 8080
CMD ["catalina.sh", "run"]
```
* We will create a pipeline type job in Jenkins
* For that we need to write a Jenkinsfile [Refer Here](https://github.com/devops-easy/game-of-life)

```
pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }
    stages {
        stage('Clone the code') {
            steps {
                git 'https://github.com/wakaleo/game-of-life'
            }
        }
        stage('Build') {
            steps {
               sh script: 'mvn clean package'
            }
        }
        stage('Docker Build') {
            steps {
               sh script: 'sudo docker image build -t <repo_name>/<image_name>:v${VERSION} .'
            }
        }
        stage('Docker Push') {
            steps {
               script {
                   withCredentials([string(credentialsId: 'dockerhub_id', variable: 'dockerhub_pass')]) {
                     sh 'sudo docker login -u <username> -p ${dockerhub_pass}'
                     sh 'sudo docker push <repo-name>/<image_name>:v${VERSION}'
                   }
               }
            }
        }
        stage('Deploy to k8s') {
            steps {
               sshagent(['k8s']) {
                 sh "scp -o StrictHostKeyChecking=no deploy.yaml ubuntu@34.215.143.119:/home/ubuntu"
                 script {
                  sh "ssh ubuntu@34.215.143.119 kubectl apply -f ."
               }
        }   
            }
        }
    }
}

```
* Kubernetes Deployment & Service File

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: gameoflife-app
  namespace: default
  labels:
    app: gameoflife-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gameoflife-app
  template:
    metadata:
      labels:
        app: gameoflife-app
    spec:
      containers:
      - name: gameoflife-app
        image: "<repo_name>/<image_name>:v1"
        ports:
          - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: gameoflife-app
  namespace: default
spec:
  selector:
    app: gameoflife-app
  type: NodePort
  ports:
  - name: http
    targetPort: 8080
    port: 80
```