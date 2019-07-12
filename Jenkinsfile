pipeline {
    environment {
      DOCKER = credentials('docker-hub')
    }
  agent any
  stages {
// Building your Test Images
    stage('BUILD') {
      parallel {
        stage('Express Image') {
          steps {
            sh 'docker build -f express-image/Dockerfile \
            -t nodeapp-dev:trunk .'
          }
        }
        stage('Test-Unit Image') {
          steps {
            sh 'docker build -f testing-image/Dockerfile \
            -t testing-image:latest .'
          }
        }
      }
      post {
        failure {
            echo 'I failed :('
// Uncomment this lines for email notifications on failure
//          mail(from: "jenkins-bot@example.com",
//           to: "devops@example.com",
//           subject: "This build failed! ${env.BUILD_TAG}",
//           body: "Check the failure ${env.BUILD_URL}")
        }
      }
    }
// Performing Software Tests
    stage('TEST') {
      parallel {
        stage('Mocha Tests') {
          steps {
            sh 'docker run --name nodeapp-dev --network="bridge" -d \
            -p 9000:9000 nodeapp-dev:trunk'
            sh 'docker run --name testing-image -v $PWD:/JUnit --network="bridge" \
            --link=nodeapp-dev -d -p 9001:9000 \
            testing-image:latest'
          }
        }
        stage('Quality Tests') {
          steps {
            sh 'docker login --username $DOCKER_USER --password $DOCKER_PWD'
            sh 'docker tag nodeapp-dev:trunk dockersp/nodeapp-dev:latest'
            sh 'docker push dockersp/nodeapp-dev:latest'
          }
        }
      }
      post {
        success {
            echo 'Success!'
// Uncomment this lines for email notifications on success
//            mail(from: "jenkins-bot@example.com",
//             to: "QA-testing@example.com",
//             subject: "New test image available ${env.BUILD_TAG}",
//             body: "Please review")
        }
        unstable {
            echo 'I am unstable'
// Uncomment this lines for email notifications when marked as unstable (failed tests)
//            mail(from: "jenkins-bot@example.com",
//             to: "QA-testing@example.com",
//             subject: "Unstable Test Results ${env.BUILD_TAG}",
//             body: "The ${env.JOB_NAME} Project had an unstable test result \
//              ${env.BUILD_URL} Branch: ${env.GIT_BRANCH} Commit: ${env.GIT_COMMIT}")
        }
        failure {
            echo 'I failed :('
// Uncomment this lines for email notifications on failure
//            mail(from: "jenkins-bot@example.com",
//             to: "devops@example.com",
//             subject: "Test Stage failed! ${env.BUILD_TAG}",
//             body: "Check the failure ${env.BUILD_URL}")
        }
      }
    }
// Deploying your Software
    stage('DEPLOY') {
          when {
           branch 'master'  //only run these steps on the master branch
          }
            steps {
                    retry(3) {
                        timeout(time:10, unit: 'MINUTES') {
                            sh 'docker tag nodeapp-dev:trunk dockersp/nodeapp-prod:latest'
                            sh 'docker push dockersp/nodeapp-prod:latest'
                            sh 'docker save dockersp/nodeapp-prod:latest | gzip > nodeapp-prod-golden.tar.gz'
                        }
                    }

            }
            post {
                failure {
                    sh 'docker stop nodeapp-dev testing-image'
                    sh 'docker system prune -f'
                    deleteDir()
                }
            }
    }
// JUnit reports and artifacts saving
    stage('REPORTS') {
      steps {
        junit 'reports.xml'
        archiveArtifacts(artifacts: 'reports.xml', allowEmptyArchive: true)
        archiveArtifacts(artifacts: 'nodeapp-prod-golden.tar.gz', allowEmptyArchive: true)
      }
    }
// Doing containers clean-up to avoid conflicts in future builds
    stage('CLEAN-UP') {
      steps {
        sh 'docker stop nodeapp-dev testing-image'
        sh 'docker system prune -f'
        deleteDir()
      }
    }
  }
}
//Testing Pipeline
