pipeline {
  agent { 
    label 'nodejs-agent'
  }
  
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    timeout(time: 15, unit: 'MINUTES')
    timestamps()
  }

  environment {
    APP_NAME = "demo-app"
    BRANCH_NAME = "${env.GIT_BRANCH.replace('origin/', '')}"
    TEAM_EMAIL = 'kousssougboss@gmail.com'
  }

  stages {
    // Étape 1: Préparation et installation
    stage('Checkout & Setup') {
      steps {
        checkout scm
        sh '''
          echo "📦 Installation des dépendances..."
          npm install
          echo "✅ Dépendances installées"
        '''
      }
    }

    // Étape 2: Qualité de code (séquentiel pour éviter les conflits)
    stage('Code Quality') {
      steps {
        sh '''
          echo "🔍 Exécution du linting..."
          npm run lint || { echo "❌ Linting échoué"; exit 1; }
          echo "✅ Linting réussi"
          
          echo "🧪 Exécution des tests unitaires..."
          npm test || { echo "❌ Tests échoués"; exit 1; }
          echo "✅ Tests réussis"
        '''
      }
      post {
        failure {
          emailext body: "Échec de la qualité de code dans le build ${env.BUILD_NUMBER}\n${env.BUILD_URL}",
                  subject: "FAILED: Code Quality - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                  to: "${TEAM_EMAIL}"
        }
      }
    }

    // Étape 3: Build Docker (uniquement pour develop)
    stage('Build Docker Image') {
      when { branch 'develop' }
      steps {
        sh '''
          echo "🐳 Construction de l'image Docker..."
          
          # Vérifier si Docker est disponible
          if ! command -v docker &> /dev/null; then
            echo "⚠️ Docker non trouvé, installation..."
            apk add --no-cache docker-cli
          fi
          
          # Build de l'image
          docker build -t ${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER} . || {
            echo "❌ Échec du build Docker"
            exit 1
          }
          
          echo "✅ Image construite: ${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER}"
          
          # Optionnel: afficher les images disponibles
          docker images | grep ${APP_NAME} || true
        '''
      }
      post {
        failure {
          emailext body: "Échec du build Docker dans le build ${env.BUILD_NUMBER}\n${env.BUILD_URL}",
                  subject: "FAILED: Docker Build - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                  to: "${TEAM_EMAIL}"
        }
      }
    }

    // Étape 4: Déploiement Staging (simulé)
    stage('Deploy to Staging') {
      when { branch 'develop' }
      steps {
        sh '''
          echo "🚀 Déploiement simulé sur l'environnement de staging"
          echo "Image utilisée: ${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER}"
          echo "Branche: ${BRANCH_NAME}"
          echo "Build: ${BUILD_NUMBER}"
          
          # Simulation du déploiement
          sleep 2
          echo "✅ Déploiement staging simulé avec succès"
        '''
      }
    }

    // Étape 5: Validation manuelle pour la production
    stage('Production Approval') {
      when { branch 'develop' }
      steps {
        script {
          def userInput = input(
            message: 'Déployer cette version en production?',
            ok: 'Confirmer le déploiement',
            parameters: [
              choice(choices: ['Oui', 'Non'], description: 'Confirmer le déploiement', name: 'DEPLOY_CHOICE')
            ]
          )
          
          if (userInput != 'Oui') {
            error('Déploiement annulé par l\'utilisateur')
          }
        }
      }
    }

    // Étape 6: Déploiement Production (simulé)
    stage('Deploy to Production') {
      when { 
        anyOf { 
          branch 'main'
          branch 'develop'  // Après approbation
        }
      }
      steps {
        sh '''
          echo "🚀 DÉPLOIEMENT EN PRODUCTION (SIMULÉ)"
          echo "=================================="
          echo "Application: ${APP_NAME}"
          echo "Version: ${BRANCH_NAME}-${BUILD_NUMBER}"
          echo "Timestamp: $(date)"
          echo "=================================="
          
          # Simulation du déploiement production
          sleep 3
          echo "✅ DÉPLOIEMENT PRODUCTION TERMINÉ AVEC SUCCÈS"
        '''
      }
    }
  }

  post {
    always {
      sh '''
        echo "🧹 Nettoyage des artefacts temporaires..."
        # Nettoyage optionnel des images Docker anciennes
        docker image prune -f --filter "until=24h" || true
        echo "✅ Pipeline terminée"
      '''
    }
    
    success {
      echo "👉 SUCCÈS: ${env.BUILD_URL}"
      emailext body: """
        🎉 Build réussi avec succès!
        
        Détails du build:
        - Application: ${APP_NAME}
        - Branche: ${BRANCH_NAME}
        - Build #${BUILD_NUMBER}
        - URL: ${env.BUILD_URL}
        
        Consultez les logs pour plus de détails.
      """,
      subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
      to: "${TEAM_EMAIL}"
    }
    
    failure {
      echo "❌ ÉCHEC: Veuillez vérifier les logs"
      emailext body: """
        ❌ Échec du pipeline!
        
        Détails de l'échec:
        - Application: ${APP_NAME}
        - Branche: ${BRANCH_NAME}
        - Build #${BUILD_NUMBER}
        - URL des logs: ${env.BUILD_URL}
        
        Veuillez consulter les logs pour identifier le problème.
      """,
      subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
      to: "${TEAM_EMAIL}"
    }
    
    unstable {
      echo "⚠️ Pipeline instable - des tests ont échoué"
      emailext body: """
        ⚠️ Pipeline instable détecté
        
        Certains tests ont échoué mais le build continue:
        - Application: ${APP_NAME}
        - Branche: ${BRANCH_NAME}
        - Build #${BUILD_NUMBER}
        - URL: ${env.BUILD_URL}
        
        Veuillez vérifier les résultats des tests.
      """,
      subject: "⚠️ UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
      to: "${TEAM_EMAIL}"
    }
  }
}