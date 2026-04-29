pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  stages {

    stage('Checkout') {
      steps {
        sh 'echo passed'
        git branch: 'main', url: 'https://github.com/iam-Rashmi-Patel/springboot-jenkins-argocd-pipeline.git'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'mvn clean package'
      }
    }

    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://52.66.4.158:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }

    stage('Build and Push Docker Image') {
      steps {
        sh '''
          apt-get update
          
          apt-get install -y awscli

          aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
          
          docker build -t $ECR_REPO:${BUILD_NUMBER} .
          
          docker push $ECR_REPO:${BUILD_NUMBER}
        '''
      }
    }

    stage('Update Deployment File') {
      environment {
        GIT_REPO_NAME = "springboot-app-manifests"
        GIT_USER_NAME = "iam-Rashmi-Patel"
      }
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          sh '''
            # Clone manifests repo
            git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
            cd ${GIT_REPO_NAME}

            # Set Git identity
            git config user.email "278338078+iam-Rashmi-Patel@users.noreply.github.com"
            git config user.name "Rashmi"

            # Update image tag
            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml

            # Commit and push changes
            git add deployment.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit"
            
            git push origin main
          '''
        }
      }
    }

  }
}