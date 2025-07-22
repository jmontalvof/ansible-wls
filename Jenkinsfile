pipeline {
  agent any

  parameters {
    choice(name: 'ENTORNO', choices: ['pre', 'pro'], description: 'Selecciona el entorno')
    string(name: 'APP_NAME', defaultValue: 'pedidos-api', description: 'Nombre de la aplicación')
    string(name: 'APP_VERSION', defaultValue: '1.0.0', description: 'Versión de la aplicación')
  }

  environment {
    NEXUS_URL = "http://172.22.25.66:8081/repository/maven-releases2"
  }

  stages {

    stage('PREPARACIÓN') {
      steps {
        sshagent(['clave-jenkins']) {
          sh """
            ansible-playbook -i inventory/${params.ENTORNO}.yml playbooks/prepare.yml \\
              --extra-vars @vars/${params.APP_NAME}.yml \\
              --extra-vars "app_name=${params.APP_NAME} app_version=${params.APP_VERSION} nexus_url=${NEXUS_URL}"
          """
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
            sh """
              ansible-playbook -i inventory/${params.ENTORNO}.yml playbooks/deploy.yml \\
                --extra-vars @vars/${params.APP_NAME}.yml \\
                --extra-vars "wl_user=${WL_USER} wl_pass=${WL_PASS} \\
                              app_name=${params.APP_NAME} app_version=${params.APP_VERSION} \\
                              nexus_url=${NEXUS_URL} nexus_user=${NEXUS_USER} nexus_pass=${NEXUS_PASS}"
            """
          }
        }
      }
    }

    stage('POST') {
      steps {
        sshagent(['clave-jenkins']) {
          sh """
            ansible-playbook -i inventory/${params.ENTORNO}.yml playbooks/postdeploy.yml \\
              --extra-vars @vars/${params.APP_NAME}.yml
          """
        }
      }
    }
  }
}

