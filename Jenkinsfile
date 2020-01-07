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

        stage('CheckStyle') {
          agent {
            docker {
              image 'maven:3.6.0-jdk-8-alpine'
              args '-v /root/.m2/repository:/root/.m2/repository'
              reuseNode true
            }

          }
          steps {
            sh ' mvn checkstyle:checkstyle'
            step([$class: 'CheckStylePublisher',
                               //canRunOnFailed: true,
                               defaultEncoding: '',
                               healthy: '100',
                               pattern: '**/target/checkstyle-result.xml',
                               unHealthy: '90',
                               //useStableBuildAsReference: true
                              ])
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
    SONARQUBE_URL = 'http://192.168.99.100'
    SONARQUBE_PORT = '9000'
  }
  options {
    skipDefaultCheckout()
  }
}
