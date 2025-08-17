pipeline {
  agent any

  environment {
    // Change these to your Docker Hub (or ECR) details
    REGISTRY = 'docker.io'
    IMAGE_NAME = 'your-dockerhub-username/java-static-demo'
    DOCKERHUB_CREDS = 'dockerhub-creds' // Jenkins Credentials ID (Username/Password)
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B -DskipTests package'
      }
      post {
        success { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
      }
    }

    stage('Unit Tests') {
      steps {
        sh 'mvn -B test'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          def tag = "${env.BUILD_NUMBER}"
          sh "docker build -t ${IMAGE_NAME}:${tag} ."
          sh "docker tag ${IMAGE_NAME}:${tag} ${IMAGE_NAME}:latest"
        }
      }
    }

    stage('Push Image') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDS, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
            sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
            sh "docker push ${IMAGE_NAME}:latest"
          }
        }
      }
    }

    stage('Deploy (Optional)') {
      when { expression { return params.DEPLOY_TO_LOCAL == true } }
      steps {
        sh '''
          docker rm -f java-static-demo || true
          docker run -d --name java-static-demo -p 8080:8080 ${IMAGE_NAME}:latest
        '''
      }
    }
  }

  parameters {
    booleanParam(name: 'DEPLOY_TO_LOCAL', defaultValue: false, description: 'Run container on the Jenkins agent after build')
  }

  post {
    always {
      cleanWs()
    }
  }
}
