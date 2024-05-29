pipeline {
  agent any

  tools {
    maven "M3"
  }

  environment {
    DOCKER_CREDENTIALS_ID = 'dockerhub-token'  // Assuming you have configured credentials with this ID
    DOCKER_IMAGE = 'ash2code/ecommerce-app'
    SLACK_CHANNEL = '#jenkins-demo'
    SLACK_CREDENTIALS_ID = 'slack-token'
    KUBECONFIG_CREDENTIALS_ID = 'kubernetes-key'
  }

  stages {
    stage('git-checkout') {
      steps {
        git 'https://github.com/ash2code/EcommerceApp.git'
      }
    }
    stage('maven-build') {
      steps {
        dir('EcommerceApp') {
          sh "mvn clean install"
        }
      }
    }
    stage('sonarqube-scan') {
      steps {
        dir('EcommerceApp') {
          withSonarQubeEnv('sonarqube-server') {
            sh """
            mvn clean verify sonar:sonar \
              -Dsonar.projectKey=Java-Ecommerce \
              -Dsonar.projectName="Java-Ecommerce" \
              -Dsonar.host.url=http://54.86.191.200:9000 \
              -Dsonar.login=sqp_5608395147c9217e580cdb2791ec957d68939c27
            """
          }
        }
      }
    }
    stage('docker-build') {
      steps {
        dir('EcommerceApp') {
          sh "docker build -t \$DOCKER_IMAGE ."
        }
      }
    }
    stage('docker-login') {
      steps {
        withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
          sh "docker login -u \$DOCKER_USERNAME --password-stdin < \$DOCKER_PASSWORD"
        }
      }
    }
    stage('docker-push') {
      steps {
        sh "docker push \$DOCKER_IMAGE"
      }
    }
    stage('kubernetes-deploy') {
      steps {
        withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
          sh """
          kubectl apply -f Ecommerce-deploysvc.yaml
          """
        }
      }
    }
  }

  post {
    success {
      slackSend(
        channel: SLACK_CHANNEL,
        color: 'good',
        message: "Pipeline '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) passed successfully!"
      )
    }
    failure {
      slackSend(
        channel: SLACK_CHANNEL,
        color: 'danger',
        message: "Pipeline '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) failed!"
      )
    }
  }
}
