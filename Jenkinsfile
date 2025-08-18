pipeline {
    agent any

    environment {
        GIT_REPO = "https://github.com/PansuraGit/Demo2.git"
        BRANCH = "master"
        DOCKER_IMAGE = "pansura/java-app"
        SONARQUBE_SERVER = "sonarqube-server"   // Configure in Jenkins -> Manage Jenkins -> Configure System
        SONARQUBE_CREDENTIALS = "sonarqube-token" // Jenkins credential ID
        DOCKER_CREDENTIALS = "dockerhub-credentials" // Jenkins credential ID
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                  trivy image --exit-code 0 --severity LOW,MEDIUM,HIGH,CRITICAL ${DOCKER_IMAGE}:${BUILD_NUMBER}
                  trivy image --exit-code 1 --severity CRITICAL ${DOCKER_IMAGE}:${BUILD_NUMBER} || true
                """
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh """
                          echo "$PASS" | docker login -u "$USER" --password-stdin
                          docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    sh """
                      docker stop java-app || true
                      docker rm java-app || true
                      docker run -d --name java-app -p 3000:8080 ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }
    }
}

