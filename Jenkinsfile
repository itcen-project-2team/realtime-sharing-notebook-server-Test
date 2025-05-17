pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-cred'
    IMAGE_NAME = 'visionn7111/whiteboard-server'
    SERVER_IP = '10.0.2.179'
  }

  stages {

    stage('Clone') {
      steps {
        git url: 'https://github.com/itcen-project-2team/realtime-sharing-notebook-server-Test.git', branch: 'main'
      }
    }

    stage('Build JAR') {
      steps {
        sh './gradlew clean build'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $IMAGE_NAME .'
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: "${DOCKERHUB_CREDENTIALS}",
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push $IMAGE_NAME
          '''
        }
      }
    }

    stage('Deploy to WAS Server') {
      steps {
        sshagent(credentials: ['webserver-ssh-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ubuntu@$SERVER_IP '
              if [ ! -d ~/whiteboard-backend ]; then
                git clone https://github.com/itcen-project-2team/realtime-sharing-notebook-server-Test.git ~/whiteboard-backend
              fi

              cd ~/whiteboard-backend

              git pull

              if [ ! -f docker-compose.yml ]; then
                echo "docker-compose.yml 파일이 없습니다. 배포 중단." && exit 1
              fi

              docker-compose down || true
              docker-compose pull
              docker-compose up -d --build
            '
          """
        }
      }
    }
  }
}
