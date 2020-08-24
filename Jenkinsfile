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

  }
  post {
    cleanup {
      deleteDir()
    }
  }
}
