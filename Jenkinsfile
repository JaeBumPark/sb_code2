pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }
  
  environment {
    gitName = 'JaeBumPark'
    gitEmail = 'jack29@naver.com'
    gitWebaddress = 'https://github.com/JaeBumPark/sb_code'
    gitSshaddress = 'git@github.com/JaeBumPark/sb_code'
    gitCredential = 'git_cre' // github credential 생성시의 ID
    dockerHubRegistry = 'kyontoki/sbimage'
    dockerHubRegistryCredential = 'docker_cre' // // docker credential 생성시의 ID
  }

    stages {
    stage('Checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
      }
      post {
        failure {
          echo 'Repository clone failure'
        }
        success {
          echo 'Repository clone success'
        }
      }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어있어야 함.
      }
      post {
        failure {
          echo 'maven build failure'
        }
        success {
          echo 'maven build success'
        }
      }
    }
    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."
        // oolralra/sbimage:4 이런식으로 빌드가 될것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수.
      }
      post {
        failure {
          echo 'docker image build failure'
        }
        success {
          echo 'docker image build success'
        }
      }
    }
    stage('docker image push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
        }

      }
      post {
        failure {
          echo 'docker image push failure'
          sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker image rm -f ${dockerHubRegistry}:latest"
        }
        success {
          sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker image rm -f ${dockerHubRegistry}:latest"  
          echo 'docker image push success'
        }
      }
    }
    stage('k8s manifest file update') {
      steps {
        git credentialsId: githubCredential,
            url: gitWebaddress,
            branch: 'main'
        
        // 이미지 태그 변경 후 메인 브랜치에 푸시
        sh "git config --global user.email ${gitEmail}"
        sh "git config --global user.name ${gitName}"
        sh "sed -i 's@${dockerHubRegistry}:.*@${dockerHubRegistry}:${currentBuild.number}@g' deploy/sb-deploy.yml"
        sh "git add ."
        sh "git commit -m 'fix:${dockerHubRegistry} ${currentBuild.number} image versioning'"
        sh "git branch -M main"
        sh "git remote remove origin"
        sh "git remote add origin ${gitSshaddress}"
        sh "git push -u origin main"

      }
      post {
        failure {
          echo 'k8s manifest file update failure'
        }
        success {
          echo 'k8s manifest file update success'  
        }
      }
    }

  }
}
