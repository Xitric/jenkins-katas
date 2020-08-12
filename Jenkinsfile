pipeline {
  agent any
  stages {
    stage('Clone down') {
        //agent {
        //    label 'host'
        //}
        steps {
            stash excludes: '.git', name: 'code'
        }
    }
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
          options {
              skipDefaultCheckout true
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls'
            deleteDir()
            sh 'ls'
          }
        }

      }
    }

  }
}
