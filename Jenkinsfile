pipeline {
  agent any

  environment {
    ACCOUNT_ID = "330167817533"
    REGION = "ap-south-1"
    REPO = "330167817533.dkr.ecr.${REGION}.amazonaws.com"
    TAG = "${BUILD_NUMBER}"
  }

  stages {

    stage('Clone') {
      steps {
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-creds', url: 'https://github.com/pravinksavant/example-voting-app.git']])
      }
    }

    stage('Build Images') {
      steps {
        sh '''
        docker build -t $REPO/vote:$TAG ./vote
        docker build -t $REPO/result:$TAG ./result
        docker build -t $REPO/worker:$TAG ./worker
        '''
      }
    }

    stage('Login to ECR') {
      steps {
        sh '''
        aws ecr get-login-password --region $REGION \
        | docker login --username AWS --password-stdin $REPO
        '''
      }
    }

    stage('Push Images') {
      steps {
        sh '''
        docker push $REPO/vote:$TAG
        docker push $REPO/result:$TAG
        docker push $REPO/worker:$TAG
        '''
      }
    }

    stage('Deploy to EKS') {
      steps {
        sh '''
        kubectl set image deployment/vote vote=$REPO/vote:$TAG
        kubectl set image deployment/result result=$REPO/result:$TAG
        kubectl set image deployment/worker worker=$REPO/worker:$TAG
        '''
      }
    }
  }
}
