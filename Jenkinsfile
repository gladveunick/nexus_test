pipeline {
    agent any
        
    tools {
        maven "M3"
    }

    environment {
        def mvn = tool 'M3'

        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://localhost:8081"
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
            filesByGlob = findFiles(glob: "target/*.${packaging}")
            // Afficher quelques informations sur l'artifact trouvé
            echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
            // Extraire le chemin du fichier trouvé
            artifactPath = filesByGlob[0].path
            // Affecter une réponse booléenne vérifiant si le nom de l'artifact existe
            artifactExists = fileExists(artifactPath)

            if (artifactExists) {
                echo "*** File: ${artifactPath}, group: ${groupId}, packaging: ${packaging}, version ${version}"

                // Téléverser l'artifact vers Nexus
                nexusArtifactUploader(
                    nexusVersion: NEXUS_VERSION,
                    protocol: NEXUS_PROTOCOL,
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


