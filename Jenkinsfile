pipeline {
  agent {
  docker {
    image 'sridhar145/maven-docker:latest'
    args '''
      --user root
      -v /var/run/docker.sock:/var/run/docker.sock
      -v /var/lib/jenkins/.m2:/root/.m2
      -v /var/lib/jenkins/.sonar:/root/.sonar
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

    MAVEN_OPTS      = "-Dmaven.repo.local=/root/.m2/repository"
    SONAR_USER_HOME = "/root/.sonar"
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
    withCredentials([usernamePassword(
      credentialsId: 'dockerhub-password',
      usernameVariable: 'DOCKER_USER',
      passwordVariable: 'DOCKER_TOKEN'
    )]) {
      sh '''
        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
      '''
      }
    }
  }

   stage('Update Kubernetes Manifest') {
  steps {
    dir(env.WORKSPACE) {
      sh '''
        # Fix Git safety issue
        git config --global --add safe.directory "$PWD"

        # Ensure we are on main branch (FIX for detached HEAD)
        git fetch origin main
        git checkout main

        git status

        sed -i "s|image:.*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|" manifests/deployment.yaml

        git config user.name "sridhar8642"
        git config user.email "sridhareswar3@gmail.com"

        git add manifests/deployment.yaml
        git commit -m "Update image to ${IMAGE_TAG}" || echo "No changes to commit"

        git push https://${GIT_CREDS_USR}:${GIT_CREDS_PSW}@github.com/sridhar8642/three-tier-backend.git main
      '''
    }
  }
}


  }
}
