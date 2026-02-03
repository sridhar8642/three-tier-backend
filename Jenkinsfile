pipeline {

  agent {
    docker {
      image 'maven:3.9.12-eclipse-temurin-21'
      args '-v /var/run/docker.sock:/var/run/docker.sock'
      reuseNode true
    }
  }

  environment {
    DOCKERHUB_USER = 'YOUR_DOCKERHUB_USERNAME'
    IMAGE_NAME = 'three-tier-backend'
    IMAGE_TAG = "${BUILD_NUMBER}"
    SONAR_HOST = 'http://SONAR_EC2_IP:9000'
    SONAR_TOKEN = credentials('sonar-token')
    DOCKER_PASS = credentials('dockerhub-password')
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build with Maven') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        sh """
          mvn sonar:sonar \
          -Dsonar.projectKey=three-tier-backend \
          -Dsonar.host.url=${SONAR_HOST} \
          -Dsonar.login=${SONAR_TOKEN}
        """
      }
    }

    stage('Docker Build') {
      steps {
        sh """
          docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
        """
      }
    }

    stage('Docker Push') {
      steps {
        sh """
          echo ${DOCKER_PASS} | docker login -u ${DOCKERHUB_USER} --password-stdin
          docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
        """
      }
    }

    stage('Update Kubernetes Manifest') {
      steps {
        sh '''
          sed -i "s|image:.*|image: '"${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"'|" manifests/deployment.yaml

          git config user.name "jenkins"
          git config user.email "jenkins@ci.local"

          git add manifests/deployment.yaml
          git commit -m "Update image to ${IMAGE_TAG}"
          git push origin main
        '''
      }
    }
  }
}
