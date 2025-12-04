pipeline {
    agent any
  
    stages {
        stage('Recuperation du code') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/jacemShady/projet-deveops.git'
            }
        }
        
        stage('Compilation Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Creation image Docker') {
            steps {
                sh 'docker build -t slm334/studentmanagement .'
            }
        }
        
        stage('Publication sur Docker Hub') {
            steps {
                sh 'docker login -u slm334 -p dckr_pat_VotreToken'
                sh 'docker push slm334/studentmanagement'
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
            echo 'Pipeline execute avec succes'
        }
        failure {
            echo 'Echec du pipeline'
        }
    }
}
