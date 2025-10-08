pipeline {
  agent any
  triggers { githubPush() }

  environment {
    DOCKER_IMAGE_NAME = 'calculator'
    DOCKERHUB_REPO    = 'shreyas0s'                 // DockerHub namespace
    DOCKERHUB_CRED    = 'DockerHubCred'         // credentialsId (username/password)
    GIT_REPO_URL      = 'https://github.com/Shreyas0S/Test_MiniProject.git'
    GIT_BRANCH        = 'main'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker --version'
        // build and tag locally
        // use `buildx build --load` so the built image is loaded into the local docker
        // image store (so the separate `docker push` step can find and push it).
        sh "docker build -t ${DOCKERHUB_REPO}/${DOCKER_IMAGE_NAME}:latest ."
      }
    }

    stage('Push Docker Image') {
      steps {
        // login using Jenkins credentials and push
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${DOCKERHUB_REPO}/${DOCKER_IMAGE_NAME}:latest
          '''
        }
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        ansiblePlaybook(playbook: 'deploy.yml', inventory: 'inventory')
      }
    }
  }

  post {
    always { cleanWs() }
  }
}
