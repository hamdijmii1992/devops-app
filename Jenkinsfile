pipeline {
  agent any
  stages {
    stage('SCM') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      parallel {
        stage('Compile') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          steps {
            sh ' mvn clean compile'
          }
        }

        

      }
    }

    stage('Integration Tests') {
      agent {
        docker {
          image 'maven:3.6.0-jdk-8-alpine'
          args '-v /root/.m2/repository:/root/.m2/repository'
          reuseNode true
        }

      }
      when {
        anyOf {
          branch 'master'
          branch 'develop'
        }

      }
      post {
        always {
          junit 'target/failsafe-reports/**/*.xml'
        }

        success {
          stash(name: 'artifact', includes: 'target/*.war')
          stash(name: 'pom', includes: 'pom.xml')
          archiveArtifacts 'target/*.war'
        }

      }
      steps {
        sh 'mvn verify -Dsurefire.skip=true'
      }
    }

    stage('Code Quality Analysis') {
      post {
        always {
          recordIssues(aggregatingResults: true, tools: [javaDoc(), checkStyle(pattern: '**/target/checkstyle-result.xml'), findBugs(pattern: '**/target/findbugsXml.xml', useRankAsPriority: true), pmdParser(pattern: '**/target/pmd.xml')])
        }

      }
      parallel {
        

        stage('SonarQube') {
          agent {
            docker {
              image 'maven:3.6.3-ibmjava-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          steps {
            sh " mvn -X -e install sonar:sonar -Dsonar.host.url=$SONARQUBE_URL:$SONARQUBE_PORT"
          }
        }

      }
    }

  }
  environment {
    NEXUS_VERSION = 'nexus3'
    NEXUS_PROTOCOL = 'http'
    NEXUS_URL = 'ec2-52-212-29-159.eu-west-1.compute.amazonaws.com:8081'
    NEXUS_REPOSITORY = 'maven-snapshots'
    NEXUS_CREDENTIAL_ID = 'nexus-credentials'
    SONARQUBE_URL = 'http://172.19.0.2'
    SONARQUBE_PORT = '9000'
  }
  options {
    skipDefaultCheckout()
  }
}
