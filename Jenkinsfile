pipeline {
  agent any

  parameters {
    string(name: 'APP_NAME', defaultValue: 'pedidos-api', description: 'Nombre de la aplicación WAR')
    string(name: 'APP_VERSION', defaultValue: '1.0.0', description: 'Versión del artefacto WAR')
    choice(name: 'ENTORNO', choices: ['pre', 'prod'], description: 'Inventario de entorno')
  }

  environment {
    NEXUS_URL = 'http://192.168.1.131:8081/repository/maven-releases'
  }

  stages {

    stage('PREPARE') {
      steps {
        sshagent(['clave-jenkins']) {
          withCredentials([
            usernamePassword(credentialsId: 'nexus-cred', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')
          ]) {
            sh '''
              export NEXUS_USER=${NEXUS_USER}
              export NEXUS_PASS=${NEXUS_PASS}
              ansible-playbook -vvv -i inventory/${ENTORNO}.yml playbooks/prepare.yml \
                --extra-vars "app_name=${APP_NAME} app_version=${APP_VERSION}"
            '''
          }
        }
      }
    }

    stage('DEPLOY') {
      steps {
        sshagent(['clave-jenkins']) {
          withCredentials([
            usernamePassword(credentialsId: 'weblogic-user', usernameVariable: 'WL_USER', passwordVariable: 'WL_PASS'),
            usernamePassword(credentialsId: 'nexus-cred', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')
          ]) {
            sh '''
              export NEXUS_USER=${NEXUS_USER}
              export NEXUS_PASS=${NEXUS_PASS}
              export WL_USER=${WL_USER}
              export WL_PASS=${WL_PASS}
              ansible-playbook -vvv -i inventory/${ENTORNO}.yml playbooks/deploy.yml \
                --extra-vars "app_name=${APP_NAME} app_version=${APP_VERSION} wl_user=${WL_USER} wl_pass=${WL_PASS}"
            '''
          }
        }
      }
    }

    stage('POSTDEPLOY') {
      steps {
        sshagent(['clave-jenkins']) {
          sh '''
            ansible-playbook -vvv -i inventory/${ENTORNO}.yml playbooks/postdeploy.yml \
              --extra-vars "app_name=${APP_NAME} app_version=${APP_VERSION}"
          '''
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'playbooks/logs/*.log', fingerprint: true
    }
  }
}
