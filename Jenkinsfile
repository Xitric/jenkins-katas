pipeline {
  agent any
  stages {
    stage('Parallel execution') {
      parallel {
        stage('Say hello') {
          steps {
            sh 'echo "Hello, world!"'
          }
        }

        stage('Build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
          }
        }

      }
    }

  }
}