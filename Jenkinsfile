pipeline {
  agent {any
        }
  tools {
    maven "M3"
  }

  environment {
      def mvn = tool 'M3'

      NEXUS_VERSION = "nexus3"
      NEXUS_PROTOCOL = "http"
      NEXUS_URL = "172.18.0.4:8081"
      NEXUS_REPOSITORY = "entrainement"
      NEXUS_CREDENTIAL_ID = "NEXUS_CREDENTIAL"
      ARTIFACT_VERSION = "${BUILD_NUMBER}"
  }
  
  stages {
    stage('Git Check out') {
      steps{
        checkout scm
      } 
    }

    stage('Maven build') {
      steps {
        sh "${mvn} clean package "
      }
    }

   // stage('SonarQube Analysis') {
      //steps{
        //withSonarQubeEnv('sonar-server') {
        //sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=toto-gros  -Dsonar.projectName=toto-gros"
     // }
    //}
     
  //}

  stage("publish to nexus") {
            steps {
                script {
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

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
 }
}
