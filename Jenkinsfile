pipeline {
  agent { 
    label 'slave-02'
  }

  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr:'3'))
  }

  environment {
    BUILD_ID = "${GIT_COMMIT}-${BUILD_ID}"
    GOOGLE_APPLICATION_CREDENTIALS = credentials('jenkins@exportarts-k8s.iam.gserviceaccount.com')
    TAG = "eu.gcr.io/exportarts-k8s/sample-docker-app:$GIT_COMMIT"
  }

  stages {

    stage('Docker login') {
      when {
        anyOf {
          branch 'master';
          tag "v*";
        }
      }

      steps {
        sh script: "gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS", label: "Docker Login"
        sh script: "gcloud auth configure-docker eu.gcr.io", label: "Configure Docker Credentials"
      }
    }

    stage('Build & Push') {
      when {
        anyOf {
          branch 'master';
          tag "v*";
        }
      }

      steps {
        sh script: "docker build -t $TAG .", label: "Build Docker Image"
        sh script: "docker push $TAG", label: "Push Docker Image"
      }
    }

  }
  post {
    cleanup {
      deleteDir()
    }
  }
}
