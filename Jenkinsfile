pipeline {
  agent any
  environment { MAVEN_HOME = tool 'Maven 3' }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Test') {
      steps { sh "${MAVEN_HOME}/bin/mvn -B test" }
    }
    stage('Package') {
      steps { sh "${MAVEN_HOME}/bin/mvn -B package" }
    }
    stage('Deploy') {
      steps {
        sshagent(credentials: ['gce-ssh']) {
          sh '''
            set -eux
            APP_VM_IP=""
            scp -o StrictHostKeyChecking=no target/*.jar deploy@${APP_VM_IP}:/home/deploy/app/hello.jar
            ssh -o StrictHostKeyChecking=no deploy@${APP_VM_IP} '
              pkill -f "java -jar /home/deploy/app/hello.jar" || true;
              nohup java -jar /home/deploy/app/hello.jar > /home/deploy/app/app.log 2>&1 &
              disown
            '
          '''
        }
      }
    }
  }
}
