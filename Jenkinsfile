pipeline {
    agent any
    
    
    environment {
        DOCKER_IMAGE = 'slm334/studentmanagement'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('📥 Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('🔨 Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('🧪 Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('🐋 Docker Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }
        
        stage('📤 Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                        echo \$PASS | docker login -u \$USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('🚀 Deploy') {
            steps {
                sh """
                    docker stop studentmanagement-app || true
                    docker rm studentmanagement-app || true
                    docker run -d -p 8081:8080 \
                        --name studentmanagement-app \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:latest
                """
            }
        }
    }
    
    post {
        success {
            echo "✅ SUCCESS! App: http://localhost:8081"
        }
        failure {
            echo "❌ BUILD FAILED!"
        }
        always {
            sh 'docker image prune -f'
        }
    }
}
