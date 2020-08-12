pipeline {
  agent any
  environment {
    docker_usernme = 'xitric'
  }
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
            stash excludes: '.git', name: 'code'
          }
        }
        
        stage('Test app') {
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
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }

      }
    }
    
    stage('Push Docker app') {
        options {
            skipDefaultCheckout true
        }
        environment {
            DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        steps {
            unstash 'code' //unstash the repository code
            sh 'ci/build-docker.sh'
            sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
            sh 'ci/push-docker.sh'
        }
    }

  }
  
  post {
    always {
        deleteDir()
    }
  }
}
