pipeline {
    agent any
    
    triggers {
        // Build automatique toutes les 15 minutes
        cron('H/15 * * * *')
    }
    
    tools {
        maven 'Maven'
        jdk 'JDK'
    }
    
    environment {
        MAVEN_OPTS = '-Dmaven.test.failure.ignore=true'
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
                    junit '**/target/surefire-reports/*.xml'
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
                archiveArtifacts artifacts: '**/target/*.jar', 
                                 fingerprint: true,
                                 allowEmptyArchive: true
            }
        }
        
        stage('Code Quality Analysis') {
            steps {
                echo '📊 Analyse de la qualité du code...'
                script {
                    try {
                        if (isUnix()) {
                            sh 'mvn checkstyle:checkstyle'
                        } else {
                            bat 'mvn checkstyle:checkstyle'
                        }
                    } catch (Exception e) {
                        echo "⚠️ Checkstyle non configuré, étape ignorée"
                    }
                }
            }
        }
        
        stage('Generate Reports') {
            steps {
                echo '📄 Génération des rapports...'
                script {
                    try {
                        if (isUnix()) {
                            sh 'mvn site'
                        } else {
                            bat 'mvn site'
                        }
                    } catch (Exception e) {
                        echo "⚠️ Génération de rapports échouée, étape ignorée"
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
            echo "✅ Durée: ${currentBuild.durationString}"
        }
        
        failure {
            echo '❌ =============================================='
            echo '❌ BUILD ÉCHOUÉ !'
            echo '❌ =============================================='
            echo "❌ Build #${env.BUILD_NUMBER} a échoué"
            echo "❌ Vérifiez les logs pour plus de détails"
        }
        
        unstable {
            echo '⚠️ =============================================='
            echo '⚠️ BUILD INSTABLE'
            echo '⚠️ =============================================='
            echo "⚠️ Des tests ont échoué dans le build #${env.BUILD_NUMBER}"
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