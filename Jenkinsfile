pipeline {
  agent any
  stages {
    stage('Dev Deployment') {
      steps {
        echo 'Dev Deployment Successful'
      }
    }

    stage('Smoke Test_Dev') {
      steps {
        echo 'Smoke Tests Passed'
      }
    }

  }
  environment {
    Dev = 'Dev Deployment'
    QA = 'QA Deployment'
  }
}