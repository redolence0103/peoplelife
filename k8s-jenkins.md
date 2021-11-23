# Jenkins 설치 및 운영

## jenkins 설치
- 저장소 키 다운로드
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys FCEF32E745F2C3D5 && sudo apt-get update
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo systemctl status jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

sudo usermod -a -G docker jenkins
```
- Installing Plugins/Upgrades
```
Dashboard > Plugin Manger
Maven Integration, Pipeline Maven Integration, Pipeline Utility Steps, CloudBees Docker Build and Publish plugin,
CloudBees Docker Hub/Registry Notification, Docker Pipeline, Docker, SSH Agent
```
- jenkins Global Tool Configuration 설정 (maven)
```
maven 항목에 MAVEN 설정 (설정되는 maven version 확인)
```
- ssh key 생성
```
ssh-keygen -t rsa
cat $HOME/id_rsa.pub >> $HOME/.ssh/authorized_keys
ssh -i ~/working.pem atid@218.236.22.95 sudo git -C /var/www.html 
```
## git 설정
```
git config --global user.name "redolence"
git config --global user.email "zasmin@uengine.net"
git config --global core.editor /usr/bin/vi
git push https://<GITHUB_ACCESS_TOKEN>@github.com/<GITHUB_USERNAME>/<REPOSITORY_NAME>.git
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

git -C /var/www/html pull origin master
```
## jenkins on docker 설치 및 운영
```
docker network ls
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_data:/var/jenkins_home jenkins/jenkins:lts
or
docker run -d --name jenkins --net kind -p 8080:8080 -p 50000:50000 -v jenkins_data:/var/jenkins_home jenkins/jenkins:lts
docker logs jenkins
...
Please use the following password to proceed to installation:

c3e74245a138465196fa76f3c7e6c0cd

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
...
docker volume ls

iptables -t nat -A PREROUTING -p TCP --dport 37013 -j DNAT --to-destination 172.17.0.2:8080
iptables -A FORWARD -p TCP --dport 8080 -d 172.17.0.2 -j ACCEPT
```
## jenkins pipeline for maven
```
pipeline {
    agent any
    tools {
        maven "MAVEN"
    }
    stages {
        stage('Build Maven') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-redolence', url: 'https://github.com/redolence0103/jenkins-docker-example.git']]])
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t zasmin/life:1.0 .'
            }
        }
    }
}

```
