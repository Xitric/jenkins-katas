pipeline {
  agent any // If not overridden, just run on any node
  
  environment {
    docker_username = 'xitric'
  }
  
  stages {
    stage('Clone down') {
        //agent {
        //    label 'host'
        //}
        steps {
            // Clones by default
            stash excludes: '.git', name: 'code' // Then stash on master
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
          agent { // Override the root agent any, run in Docker
            docker {
              image 'gradle:jdk11'
            }

          }
          options {
              skipDefaultCheckout true // Skip cloning
          }
          steps {
            unstash 'code' // Retrieve stashed code from master
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash excludes: '.git', name: 'code' // Put new changes into master
            //sh 'ls'
            //deleteDir()
            //sh 'ls'
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
    
    stage('Build Docker') {
        options {
            skipDefaultCheckout true
        }
        steps {
            unstash 'code' // Retrieve the recently updated files from master
            sh 'ci/build-docker.sh'
            stash name: 'dockerimg'
        }
    }
    
    stage('Push Docker') {
        when {
            branch 'master'
        }
        options {
            skipDefaultCheckout true
        }
        environment {
            // Retrieve credentials from Jenkins credential store
            // Saved as DOCKERCREDS_USR and DOCKERCREDS_PSW
            DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
        }
        steps {
            unstash 'dockerimg'
            sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
            sh 'ci/push-docker.sh'
        }
    }
    
    stage('Component test') {
        when {
            //not {
            //    branch pattern: "dev/.+", comparator: "REGEXP"
            //}
          anyOf {
            branch 'master'
            changeRequest()
          }
        }
        options {
            skipDefaultCheckout true
        }
        steps {
            // Only works because build and push are running on the same node.
            // Otherwise the Docker repository would be on a different machine.
            // We should configure this and the previous step to run on the same node.
            unstash 'dockerimg'
            sh 'ci/component-test.sh'
        }
    }

  }
  
  // Run deleteDir after every stage of this pipeline to clean up
  post {
    always {
        deleteDir()
    }
  }
}
