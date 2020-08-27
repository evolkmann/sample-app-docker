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
    GOOGLE_CLOUD_PROJECT = 'exportarts-k8s'
    GOOGLE_CLOUD_REGION = 'europe-west3'
    GOOGLE_CLOUD_ZONE = "$GOOGLE_CLOUD_REGION-a"
    GOOGLE_CLOUD_CLUSTER = 'prod'
    K8S_DEPLOY_YAML = 'kubernetes/deploy.yml'
    DOCKER_IMAGE_NAME = "eu.gcr.io/$GOOGLE_CLOUD_PROJECT/sample-docker-app"
  }

  stages {
    stage('Set Environment Variables') {
      steps {
        script {
          if (env.TAG_NAME) {
            env.DOCKER_IMAGE_TAG = env.TAG_Name
          } else {
            env.DOCKER_IMAGE_TAG = env.GIT_BRANCH + '-' + env.BUILD_NUMBER
          }
        }
      }
    }

    stage('Docker login') {
      when {
        anyOf {
          branch 'master';
          tag "*";
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
          tag "*";
        }
      }

      steps {
        sh script: "docker build -t $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG .", label: "Build Docker Image"
        sh script: "docker push $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG", label: "Push Docker Image"
      }
    }

    stage('Deploy') {
      when {
        anyOf {
          branch 'master';
          tag "*";
        }
      }

      steps {
        sh script: "gcloud config set project $GOOGLE_CLOUD_PROJECT", label: "Set Project ID"
        sh script: "gcloud config set compute/zone $GOOGLE_CLOUD_ZONE", label: "Set Zone"
        sh script: "gcloud container clusters get-credentials $GOOGLE_CLOUD_CLUSTER", label: "Get Cluster Credentials"
        sh script: """
          sed -i 's ~~IMAGE~~ $DOCKER_IMAGE_NAME:$DOCKER_IMAGE_TAG g' $K8S_DEPLOY_YAML
        """, label: "Replace with Environment Vars"
        sh script: "cat $K8S_DEPLOY_YAML", label: "Dump Deployment File"
        sh script: "kubectl apply -f $K8S_DEPLOY_YAML", label: "Deploy"
      }
    }
    

  }
  post {
    cleanup {
      deleteDir()
    }
  }
}
