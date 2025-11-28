pipeline {
    agent any
    
    triggers {
        cron('H/15 * * * *')
    }
    
    environment {
        DOCKER_IMAGE = 'slm334/studentmanagement'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📥 Récupération du code source...'
                checkout scm
            }
        }
        
        stage('Build Maven') {
            steps {
                echo '🔨 Construction du projet Maven...'
                script {
                    if (isUnix()) {
                        sh 'mvn clean package -DskipTests'
                    } else {
                        bat 'mvn clean package -DskipTests'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Exécution des tests...'
                script {
                    if (isUnix()) {
                        sh 'mvn test'
                    } else {
                        bat 'mvn test'
                    }
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '🐋 Construction de l\'image Docker...'
                script {
                    if (isUnix()) {
                        sh """
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        """
                    } else {
                        bat """
                            docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                echo '📤 Push vers Docker Hub...'
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        if (isUnix()) {
                            sh """
                                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                                docker push ${DOCKER_IMAGE}:latest
                            """
                        } else {
                            bat """
                                echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                                docker push ${DOCKER_IMAGE}:latest
                            """
                        }
                    }
                }
            }
        }
        
        stage('Deploy Container') {
            steps {
                echo '🚀 Déploiement du conteneur...'
                script {
                    if (isUnix()) {
                        sh """
                            # Arrêter et supprimer l'ancien conteneur
                            docker stop studentmanagement-app || true
                            docker rm studentmanagement-app || true
                            
                            # Lancer le nouveau conteneur
                            docker run -d -p 8081:8080 \
                                --name studentmanagement-app \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE}:latest
                            
                            # Vérifier que le conteneur démarre
                            sleep 10
                            docker ps | grep studentmanagement-app
                        """
                    } else {
                        bat """
                            docker stop studentmanagement-app || exit 0
                            docker rm studentmanagement-app || exit 0
                            docker run -d -p 8081:8080 --name studentmanagement-app ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ =============================================='
            echo '✅ BUILD ET DÉPLOIEMENT RÉUSSIS !'
            echo '✅ =============================================='
            echo "✅ Image Docker: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "✅ Application disponible sur: http://localhost:8081"
        }
        
        failure {
            echo '❌ =============================================='
            echo '❌ BUILD OU DÉPLOIEMENT ÉCHOUÉ !'
            echo '❌ =============================================='
        }
        
        always {
            echo '🧹 Nettoyage des images non utilisées...'
            script {
                if (isUnix()) {
                    sh 'docker image prune -f'
                }
            }
        }
    }
}
