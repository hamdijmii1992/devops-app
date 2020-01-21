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


      stage('Deploy Artifact To Nexus') {
   when {
    anyOf { branch 'master'; branch 'develop' }
   }
   steps {
    script {
     unstash 'pom'
     unstash 'artifact'
     // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
     pom = readMavenPom file: "pom.xml";
     // Find built artifact under target folder
     filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
     // Print some info from the artifact found
     echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
     // Extract the path from the File found
     artifactPath = filesByGlob[0].path;
     // Assign to a boolean response verifying If the artifact name exists
     artifactExists = fileExists artifactPath;
     if (artifactExists) {
      nexusArtifactUploader(
       nexusVersion: NEXUS_VERSION,
       protocol: NEXUS_PROTOCOL,
       nexusUrl: NEXUS_URL,
       groupId: pom.groupId,
       version: pom.version,
       repository: NEXUS_REPOSITORY,
       credentialsId: NEXUS_CREDENTIAL_ID,
       artifacts: [
        // Artifact generated such as .jar, .ear and .war files.
        [artifactId: pom.artifactId,
         classifier: '',
         file: artifactPath,
         type: pom.packaging
        ],
        // Lets upload the pom.xml file for additional information for Transitive dependencies
        [artifactId: pom.artifactId,
         classifier: '',
         file: "pom.xml",
         type: "pom"
        ]
       ]
      )
     } else {
      error "*** File: ${artifactPath}, could not be found";
     }
    }
   }
  }

      stage('Deploy to Staging Servers') {
   when {
    anyOf { branch 'master'; branch 'develop' }
   }
   agent {
    docker {
     image 'ahmed24khaled/ansible-management'
     args '--network=devops'
     reuseNode true
    }
   }
   steps {
    script {
     try {
     
     pom = readMavenPom file: "pom.xml"
     repoPath = "${pom.groupId}".replace(".", "/") + "/${pom.artifactId}"
     version = pom.version
     artifactId = pom.artifactId
     withEnv(["ANSIBLE_HOST_KEY_CHECKING=False", "APP_NAME=${artifactId}", "repoPath=${repoPath}", "version=${version}"]) {
     sh '''
      
        curl --silent "http://$NEXUS_URL/repository/maven-snapshots/${repoPath}/${version}/maven-metadata.xml" > tmp
       '''
     }
       } catch (Exception e) {
       echo "Caught exception: ${e}"
  }
    }
   }
  }
  }
  environment {
    NEXUS_VERSION = 'nexus3'
    NEXUS_PROTOCOL = 'http'
    NEXUS_URL = 'nexus:8081'
    NEXUS_REPOSITORY = 'maven-snapshots'
    NEXUS_CREDENTIAL_ID = 'nexus-credentials'
    SONARQUBE_URL = 'http://sonarqube'
    SONARQUBE_PORT = '9000'
  }
  options {
    skipDefaultCheckout()
  }
}
