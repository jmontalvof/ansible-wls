pipeline {
  agent any

  options {
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  parameters {
    string(name: 'APP_NAME', defaultValue: 'simple-webapp', 
           description: 'Nombre de la aplicación WAR')
    string(name: 'APP_VERSION', defaultValue: '1.0', 
           description: 'Versión del artefacto WAR')
    choice(name: 'ENVIRONMENT', choices: ['pre', 'prod'], 
           description: 'Entorno de despliegue')
  }

  environment {
    NEXUS_URL = 'http://192.168.1.131:8081/repository/maven-releases'
    LOGS_DIR = 'logs'
  }

  stages {
    stage('Initialize') {
      steps {
        script {
          // Crear directorio de logs si no existe
          sh "mkdir -p ${LOGS_DIR}"
          
          // Mostrar parámetros de ejecución
          echo """
          ========== PARÁMETROS DE DESPLIEGUE ==========
          Aplicación: ${APP_NAME}
          Versión: ${APP_VERSION}
          Entorno: ${ENVIRONMENT}
          =============================================
          """
        }
      }
    }

    stage('Prepare Environment') {
      steps {
        sshagent(['clave-jenkins']) {
          withCredentials([
            usernamePassword(credentialsId: 'nexus-cred', 
                           usernameVariable: 'NEXUS_USER', 
                           passwordVariable: 'NEXUS_PASS')
          ]) {
            sh """
              export NEXUS_USER=${NEXUS_USER}
              export NEXUS_PASS=${NEXUS_PASS}
              ansible-playbook -i inventory/${ENVIRONMENT}.yml playbooks/prepare.yml \
                --extra-vars "app_name=${APP_NAME} app_version=${APP_VERSION}"
            """
          }
        }
      }
    }

    stage('Deploy Application') {
      steps {
        sshagent(['clave-jenkins']) {
          withCredentials([
            usernamePassword(credentialsId: 'weblogic-user', 
                           usernameVariable: 'WL_USER', 
                           passwordVariable: 'WL_PASS'),
            usernamePassword(credentialsId: 'nexus-cred', 
                           usernameVariable: 'NEXUS_USER', 
                           passwordVariable: 'NEXUS_PASS')
          ]) {
            sh """
              export WL_USER=${WL_USER}
              export WL_PASS=${WL_PASS}
              export NEXUS_USER=${NEXUS_USER}
              export NEXUS_PASS=${NEXUS_PASS}
              ansible-playbook -i inventory/${ENVIRONMENT}.yml playbooks/deploy.yml \
                --extra-vars "app_name=${APP_NAME} app_version=${APP_VERSION} wl_user=${WL_USER} wl_pass=${WL_PASS}"
            """
          }
        }
      }
    }

    stage('Validate Deployment') {
      steps {
        sshagent(['clave-jenkins']) {
          sh """
            ansible-playbook -i inventory/${ENVIRONMENT}.yml playbooks/postdeploy.yml \
              --extra-vars "app_name=${APP_NAME} app_version=${APP_VERSION}"
          """
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: "${LOGS_DIR}/*.log", fingerprint: true
      cleanWs()
    }
    success {
      slackSend(color: 'good', message: "Despliegue exitoso: ${APP_NAME} v${APP_VERSION} en ${ENVIRONMENT}")
    }
    failure {
      slackSend(color: 'danger', message: "Fallo en despliegue: ${APP_NAME} v${APP_VERSION} en ${ENVIRONMENT}")
    }
  }
}
