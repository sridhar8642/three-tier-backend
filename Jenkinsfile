pipeline {
  agent {
  docker {
    image 'maven:3.9.12-eclipse-temurin-21'
    args '''
      -v /var/run/docker.sock:/var/run/docker.sock
      -v /var/lib/jenkins/.m2:/home/jenkins/.m2
      -v /var/lib/jenkins/.sonar:/home/jenkins/.sonar
    '''
    reuseNode true
    }
  }

  environment {
    // DockerHub
    DOCKERHUB_USER = 'sridhar145'
    IMAGE_NAME     = 'three-tier-backend'
    IMAGE_TAG      = "${BUILD_NUMBER}"
    DOCKER_PASS    = credentials('dockerhub-password')

    // SonarQube
    SONAR_HOST  = 'http://43.205.139.198:9000'
    SONAR_TOKEN = credentials('sonar-token')

    // GitHub credentials (PAT)
    GIT_CREDS = credentials('github-creds')

    MAVEN_OPTS = "-Dmaven.repo.local=/home/jenkins/.m2/repository"
    SONAR_USER_HOME = "/home/jenkins/.sonar"
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
        // If you have mvnw, prefer:
        // sh './mvnw clean package'
      }
    }

   stage('SonarQube Analysis') {
     steps {
    sh '''
      mvn sonar:sonar \
        -Dsonar.projectKey=sridhar_three_tier_backend \
        -Dsonar.projectName=three-tier-backend \
        -Dsonar.host.url=http://43.205.139.198:9000 \
        -Dsonar.login=$SONAR_TOKEN
    '''
    }
  }

    /*
    // OPTIONAL but RECOMMENDED
    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    */

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

          git config user.name "sridhar8642"
          git config user.email "sridhareswar3@gmail.com"

          git add manifests/deployment.yaml
          git commit -m "Update image to ${IMAGE_TAG}"

          git push https://${GIT_CREDS_USR}:${GIT_CREDS_PSW}@github.com/sridhar8642/three-tier-backend.git main
        '''
      }
    }
  }
}
