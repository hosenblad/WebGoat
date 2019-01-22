pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh '''
            echo "PATH = ${PATH}"
            echo "M2_HOME = ${M2_HOME}"
            mvn -B install '''
      }
    }
    stage ('Creating build tag') {
      steps {
            createTag nexusInstanceId: 'nx3', tagAttributesJson: '{"createdBy" : "Moose"}', tagName: 'build-125'
        }
    }
    stage ('Publishing') {
        steps {
            nexusPublisher nexusInstanceId: 'nx3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'webgoat-container/target/webgoat-container-8.0.0.M3.jar']], mavenCoordinate: [artifactId: 'fancyWidget', groupId: 'com.mycompany', packaging: 'jar', version: '1.1']]], tagName: 'build-125'
        }
    }
    stage ('Move') {
        steps {
            moveComponents destination: 'maven-test', nexusInstanceId: 'nx3', tagName: 'build-123'
        }
    }
    stage ('Delete') {
        steps {
            deleteComponents nexusInstanceId: 'nx3', tagName: 'build-120'
        }
    } 

    stage('Scan App - Build Container') {
      parallel {
        stage('IQ-BOM') {
          steps {
            nexusPolicyEvaluation(iqApplication: 'webgoat8', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
        }
        stage('Static Analysis') {
          steps {
            echo '...run SonarQube or other SAST tools here'
          }
        }
        stage('Build Container') {
          steps {
            sh '''cd webgoat-server
docker build -t webgoat/webgoat-8.0 .
                    '''
          }
        }
      }
    }
    stage('Test Container') {
      parallel {
        stage('Test Container') {
          steps {
            catchError() {
              echo 'Test Container here'
            }

          }
        }
        stage('IQ-Scan Container') {
          steps {
            sh 'docker save webgoat/webgoat-8.0 -o $WORKSPACE/webgoat.tar'
            nexusPolicyEvaluation(iqStage: 'stage-release', iqApplication: 'webgoat8')
          }
        }
      }
    }
    stage('Publish Container') {
      when {
        branch 'master'
      }
      steps {
        sh '''
                    docker tag webgoat/webgoat-8.0 mycompany.com:5000/webgoat/webgoat-8.0:8.0
                    docker push mycompany.com:5000/webgoat/webgoat-8.0
                '''
      }
    }
  }
  tools {
    maven 'M3'
  }
}