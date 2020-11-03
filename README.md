# Jenkins CICD Introduction Demo Code and Configs

Contains Jenkins code and configuration examples as used within the [Introduction to CI/CD with Jenkins](https://cloudacademy.com/course/jenkins-cicd/build-pipelines-declarative) course.

## Demo 4: Install Jenkins - EC2 Ubuntu

Install intructions for Ubuntu 18.04

```
sudo -s
apt-get update
apt-get install -y openjdk-8-jdk
java -version
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
apt-get update
apt-get install -y jenkins
```

```
systemctl status jenkins
ps -ef | grep jenkins  
ufw status
ufw allow 8080
```

```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

## Demo 5: Install Jenkins - Docker Compose

```
mkdir jenkins-docker
cd jenkins-docker
```

```
docker network create -d bridge devnetwork
```

docker-compose.yml

```
version: "3.5"
services:
    jenkins:
        container_name: jenkins-docker
        image: jenkins/jenkins:lts
        ports:
            - 8080:8080
        networks:
            - devnetwork
networks:
    devnetwork:
        name: devnetwork
```

```
docker-compose config
```

```
docker-compose up -d && docker-compose logs -f
```

```
docker ps
```

```
docker exec -it jenkins-docker bash
```

```
docker exec -it jenkins-docker /var/lib/jenkins_home/secrets/initialAdminPassword 
```

## Demo 6: Create Basic Freestyle Project

UI instructions only

## Demo 7: Create Basic Freestyle Project

UI instructions only

## Demo 8: Building a GitHub Stored Project

Refs
https://github.com/cloudacademy/react-webapp

Note: assumes yarn is installed on the server

```
yarn install
yarn build
ls -la
echo finished!!
```

## Demo 9: Build Triggers with GitHub Hooks

Documentation
https://api.github.com/meta

Refs
https://github.com/cloudacademy/react-webapp

## Demo 13: Build Pipelines - Maven

Uses provided sample pipeline within the Jenkins UI

## Demo 14: Build Pipelines - Scripted

```
def int fibonacci(int n){
    n < 2 ? n : fibonacci(n-1) + fibonacci(n-2)
}

node{
    def workspace = pwd()
    echo "workspace = ${workspace}"

    def nine = 9
    def ten = nine + 1

    stage('Calculate'){
        try{
            if(ten > nine){
                echo "${fibonacci(ten)}"
                sh returnStdout: true, script: 'script-which-doesnt-exist.sh'
            }
        }
        catch(exc){
            echo "some exception just happened!!" 
        }

    }
}
```

## Demo 15: Build Pipelines - Gradle

Refs
https://github.com/cloudacademy/devops-webapp

Website
https://crontab.guru/

```
H/5 * * * *
```

```
//START-OF-SCRIPT
node {
    def GRADLE_HOME = tool name: 'gradle-4.10.2', type: 'hudson.plugins.gradle.GradleInstallation'
    sh "${GRADLE_HOME}/bin/gradle tasks"

    stage('Clone') {
        git url: 'https://github.com/cloudacademy/devops-webapp.git'
    }

    stage('Build') {
        sh "${GRADLE_HOME}/bin/gradle build"
    }

    stage('Archive') {
        archiveArtifacts 'build/libs/*.war'
    }
}
//END-OF-SCRIPT
```

```
sudo apt-get install -y tree
cd /var/lib/jenkins/workspace/BuildJob6
ls -la
cd build
tree
```

====================

## Demo 16: Build Pipelines - Declarative

```
pipeline {
    agent {
        label 'master'
    }
    tools {
        gradle 'gradle-4.10.2'
    }
    environment {
        VERSION = "jellybean"
    }
    stages{
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: 'master']],
                    extenstions: [[$class: 'WipeWorkspace']],
                    userRemoteConfigs: [[url: 'https://github.com/cloudacademy/devops-webapp.git']]

                ])
            }
        }
        stage('Details') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
                echo "${env.VERSION}"
            }

            when {
                environment name: 'VERSION', value: 'jellybean'
            }
        }
        stage('Build') {
            steps {
                sh "gradle build"
            }
        }
    }
}
```