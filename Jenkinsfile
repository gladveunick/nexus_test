pipeline {
    agent any
        
    tools {
        maven "M3"
    }

    environment {
        def mvn = tool 'M3'

        //NEXUS_VERSION = "nexus3"
        NEXUS_VERSION = "Sonatype Nexus Repository OSS 3.65.0-02"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "127.0.0.1:8081"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexusCredential"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }
  
    stages {
        stage('Git Check out') {
            steps {
                checkout scm
            } 
        }

        stage('Maven build') {
            steps {
                bat "\"${mvn}\\bin\\mvn\" clean package"
            }
        }


        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    script {
                        def mvnHome = tool 'M3'
                        bat "\"${mvnHome}\\bin\\mvn\" clean verify sonar:sonar -Dsonar.projectKey=nexus_test -Dsonar.projectName=nexus_test -Dsonar.login=sqp_dc4e42156107de753613f546f0dae226ac000e93 -Dsonar.host.url=http://localhost:9000"
                    }
                }
            }
        }

        stage('Test Connectivity') {
            steps {
                script {
                    def nexusUrl = "http://127.0.0.1:8081"
                    def command = "curl -I ${nexusUrl}"
                    def output = bat(script: command, returnStdout: true).trim()
                    echo "Output from curl command:"
                    echo output
                }
            }
        }

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
        // stage ('Execute Ansible Play - CD'){
        //     agent {
        //         label 'ansible'
        //     }
        //     steps{
        //         script {
        //             git branch: 'feature/ansibleNexus', url: 'https://github.com/ranjit4github/Ansible_Demo_Project.git';
        //         }
        //         sh '''
        //             ansible-playbook -e vers=${BUILD_NUMBER} roles/site.yml
        //         '''
        //     }
        // }
    }
}

//         stage("publish to nexus") {
//             steps {
//                 script {
//                     // Définir manuellement les informations du POM
//                     def groupId = "sn.dev"
//                     def artifactId = "nexus_test"
//                     def version = "1.0-SNAPSHOT"
//                     def packaging = "jar" // Le packaging est généralement "jar" pour les projets Java
                    
//                     // Rechercher l'artifact construit dans le dossier cible
//                     def files = bat script: 'dir /b /s target\\*.jar', returnStdout: true
//                     def artifactPath = files.split("\\r?\\n")[0].trim()

//                     // Affecter une réponse booléenne vérifiant si le nom de l'artifact existe
//                     def artifactExists = fileExists(artifactPath)

//                     if (artifactExists) {
//                         echo "*** File: ${artifactPath}, group: ${groupId}, packaging: ${packaging}, version ${version}"

//                         // Téléverser l'artifact vers Nexus
//                         nexusArtifactUploader(
//                             nexusVersion: NEXUS_VERSION,
//                             protocol: NEXUS_PROTOCOL,
//                             nexusUrl: NEXUS_URL,
//                             groupId: groupId,
//                             version: version,
//                             repository: NEXUS_REPOSITORY,
//                             credentialsId: NEXUS_CREDENTIAL_ID,
//                             artifacts: [
//                                 // Artifacts générés tels que les fichiers .jar, .ear et .war.
//                                 [artifactId: artifactId,
//                                 classifier: '',
//                                 file: artifactPath,
//                                 type: packaging]
//                             ]
//                         )
//                     } else {
//                         error "*** File: ${artifactPath}, could not be found"
//                     }
//                 }
//             }
//         }
//     }
// }
