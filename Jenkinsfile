pipeline {
    agent any
  
    stages {
        stage('Recuperation du code') {
            steps {
                git branch: 'master', 
                    url: 'https://github.com/jacemShady/projet-deveops.git'
            }
        }
        
        stage('Compilation Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=studentmanagement -Dsonar.host.url=http://localhost:9000'
                }
            }
        }
        
        stage('Creation image Docker') {
            steps {
                sh 'docker build -t slm334/studentmanagement .'
            }
        }
        
        
        stage('Publication sur Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_TOKEN')]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push slm334/studentmanagement
                    '''
                }
            }
        }

        
        stage('Deploiement') {
            steps {
                sh 'docker stop studentmanagement-app || true'
                sh 'docker rm studentmanagement-app || true'
                sh 'docker run -d -p 8081:8080 --name studentmanagement-app slm334/studentmanagement'
            }
        }
        
    }
    
    post {
        success {
            mail to: 'guesmijacem8@gmail.com',
                 subject: 'Build Successful',
                 body: 'La build a réussi.'
        }
        failure {
            mail to: 'guesmijacem8@gmail.com',
                 subject: 'Build Failed',
                 body: 'La build a échoué.'
        }
    }

}
