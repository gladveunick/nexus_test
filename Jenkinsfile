pipeline {
    agent any
        
    tools {
        maven "M3"
    }

    environment {
        def mvn = tool 'M3'

        NEXUS_VERSION = "nexus3"
        // NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://127.0.0.1:8081"
        NEXUS_REPOSITORY = "entrainement"
        NEXUS_CREDENTIAL_ID = "NEXUS_CREDENTIAL"
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

        stage('Test Connectivity') {
            steps {
                script {
                    def nexusUrl = "http://127.0.0.1:8081/repository/entrainement/"
                    def command = "powershell.exe -Command \"(Invoke-WebRequest -Uri '${nexusUrl}').Headers\""
                    def output = bat(script: command, returnStdout: true).trim()
                    echo "Output from PowerShell command:"
                    echo output
                }
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

        stage("publish to nexus") {
            steps {
                script {
                    // Définir manuellement les informations du POM
                    def groupId = "sn.dev"
                    def artifactId = "nexus_test"
                    def version = "1.0-SNAPSHOT"
                    def packaging = "jar" // Le packaging est généralement "jar" pour les projets Java
                    
                    // Rechercher l'artifact construit dans le dossier cible
                    def files = bat script: 'dir /b /s target\\*.jar', returnStdout: true
                    def artifactPath = files.split("\\r?\\n")[0].trim()

                    // Affecter une réponse booléenne vérifiant si le nom de l'artifact existe
                    def artifactExists = fileExists(artifactPath)

                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${groupId}, packaging: ${packaging}, version ${version}"

                        // Téléverser l'artifact vers Nexus
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            // protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: groupId,
                            version: version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifacts générés tels que les fichiers .jar, .ear et .war.
                                [artifactId: artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: packaging]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
}
