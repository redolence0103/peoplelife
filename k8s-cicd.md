# Jenkins CI/CD

## cicd process 설정

1.  github 설정
   - GitHub 에 코드를 upload
2. jenkins 설정
   - Jenkins 서버에 코드를 download
3. build/compile 프로젝트 (gradle)
   - Gradle 사용 환경을 설정
4. build docker image
   - Docker Image 생성
5. Push latest docker image to dockerhub
   - 생성된 Docker image를 Docker Hub에 upload
6. Pull latest docker image from dockerhub
   - Docker image download
7. deploy deployment app into K8S cluster
   - K8S 클러스터에 App 배포
## jenkins groovy file
```
node {
    stage("Git Clone"){

        git credentialsId: 'GIT_HUB_CREDENTIALS', url: 'https://github.com/redolence0103/springboot-with-docker.git'
    }
    stage('Gradle Build') {

       sh './gradlew build'

    }
    stage("Docker build"){
        sh 'docker version'
        sh 'docker build -t sit-cicd-demo .'
        sh 'docker image list'
        sh 'docker tag sit-cicd-demo zasmin/sit-cicd-demo:1.0'
    } 
    stage("Docker Login"){
        withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]) {
            sh 'docker login -u zasmin -p $PASSWORD'
        }
    } 
    stage("Push Image to Docker Hub"){
        sh 'docker push  zasmin/sit-cicd-demo:1.0'
    }
    stage("SSH Into k8s Client") {
        def remote = [:]
        remote.name = 'K8S Client'
        remote.host = '218.236.22.90'
        remote.user = 'atid'
        remote.password = 'wjdgid0103'
        remote.allowAnyHosts = true
        
        stage('Put sit-cicd-demo-dep.yaml onto K8S Client') {
        sshPut remote: remote, from: 'sit-cicd-demo-dep.yaml', into: '.'
        } 
        stage('Deploy spring boot') {
          sshCommand remote: remote, command: "kubectl apply -f sit-cicd-demo-dep.yaml"
        }
    }
    
    
    
}
```
http://218.236.22.90:8080/
