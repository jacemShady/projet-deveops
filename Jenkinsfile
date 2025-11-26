pipeline {
    agent any
    
    triggers {
        cron('H/15 * * * *')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📥 Récupération du code source depuis Git...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Construction du projet Maven...'
                script {
                    if (isUnix()) {
                        sh 'mvn clean compile'
                    } else {
                        bat 'mvn clean compile'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Exécution des tests unitaires...'
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
                    script {
                        try {
                            junit '**/target/surefire-reports/*.xml'
                        } catch (Exception e) {
                            echo "⚠️ Pas de rapports de tests trouvés"
                        }
                    }
                }
            }
        }
        
        stage('Package') {
            steps {
                echo '📦 Packaging de l\'application...'
                script {
                    if (isUnix()) {
                        sh 'mvn package -DskipTests'
                    } else {
                        bat 'mvn package -DskipTests'
                    }
                }
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo '💾 Archivage des artefacts...'
                script {
                    try {
                        archiveArtifacts artifacts: '**/target/*.jar', 
                                         fingerprint: true,
                                         allowEmptyArchive: true
                    } catch (Exception e) {
                        echo "⚠️ Pas d'artefacts à archiver"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ =============================================='
            echo '✅ BUILD RÉUSSI !'
            echo '✅ =============================================='
            echo "✅ Build #${env.BUILD_NUMBER} terminé avec succès"
        }
        
        failure {
            echo '❌ =============================================='
            echo '❌ BUILD ÉCHOUÉ !'
            echo '❌ =============================================='
            echo "❌ Build #${env.BUILD_NUMBER} a échoué"
        }
        
        unstable {
            echo '⚠️ =============================================='
            echo '⚠️ BUILD INSTABLE'
            echo '⚠️ =============================================='
        }
        
        always {
            echo '🧹 Nettoyage de l\'espace de travail...'
            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true,
                notFailBuild: true
            )
        }
    }
}
