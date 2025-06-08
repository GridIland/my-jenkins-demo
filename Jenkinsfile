pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    timeout(time: 15, unit: 'MINUTES')
    timestamps()
  }

  environment {
    APP_NAME = "demo-app"
    BRANCH_NAME = env.BRANCH_NAME.replaceAll('/', '-')
    // Configurer dans Jenkins > System Configuration
    TEAM_EMAIL = 'kousssougboss@gmail.com'
  }

  stages {
    // Étape 1: Préparation
    stage('Checkout & Setup') {
      agent {
        dockerContainer {
          image 'icontain/jenkins-node-agent'
        }
      }
      steps {
        checkout scm
        sh 'npm install'
      }
    }

    // Étape 2: Qualité de code
    stage('Lint & Test') {
      parallel {
        stage('Linting') {
          agent {
            dockerContainer {
              image 'icontain/jenkins-node-agent'
            }
          }
          steps {
            sh 'npm run lint'
          }
          post {
            failure {
              emailext body: "Linting failed in build ${env.BUILD_NUMBER}\n${env.BUILD_URL}",
                      subject: "FAILED: Linting - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                      to: "${TEAM_EMAIL}"
            }
          }
        }
        stage('Unit Tests') {
          agent {
            dockerContainer {
              image 'icontain/jenkins-node-agent'
            }
          }
          steps {
            sh 'npm test'
          }
          post {
            failure {
              emailext body: "Unit tests failed in build ${env.BUILD_NUMBER}\n${env.BUILD_URL}",
                      subject: "FAILED: Tests - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                      to: "${TEAM_EMAIL}"
            }
          }
        }
      }
    }

    // Étape 3: Build Docker
    stage('Build Image') {
      when { branch 'develop' }
      agent {
        dockerContainer {
          image 'docker:24-cli'
        }
      }
      steps {
        sh 'docker build -t ${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER} .'
      }
    }

    // Étape 4: Déploiement Staging (simulé)
    stage('Deploy to Staging') {
      when { branch 'develop' }
      agent any
      steps {
        echo "🚀 Déploiement simulé sur staging"
        echo "Image: ${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER}"
      }
    }

    // Étape 5: Validation manuelle
    stage('Approbation Production') {
      when { branch 'develop' }
      agent none
      steps {
        input message: "Déployer en production?", ok: "Confirmer"
      }
    }

    // Étape 6: Déploiement Production (simulé)
    stage('Deploy to Production') {
      when { 
        anyOf { 
          branch 'main'
          expression { return true } // Toujours exécuté après approbation
        }
      }
      agent any
      steps {
        echo "🚀 DÉPLOIEMENT PRODUCTION SIMULÉ"
        echo "Version: ${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER}"
      }
    }
  }

  post {
    always {
      echo "✅ Pipeline terminée"
    }
    success {
      echo "👉 SUCCÈS: ${env.BUILD_URL}"
    }
    failure {
      echo "❌ ÉCHEC: Veuillez vérifier les logs"
      
    }
    unstable {
      echo "⚠️ Des tests sont instables"
    }
  }
}