
pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "169.56.94.238:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "maven-example"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    stages {
        stage("clone code") {
            steps{
                // Let's clone the source
                git 'http://169.56.94.238:8088/root/maven-app-sample.git';    
            }
        }
        stage('SonarQube analysis') {
            steps {
                // withSonarQubeEnv('My SonarQube Server') { // If you have configured more than one global server connection, you can specify its name
                //   sh "${scannerHome}/bin/sonar-scanner"
                // }
                script {
                    sh "mvn sonar:sonar \
                          -Dsonar.projectKey=maven-sample \
                          -Dsonar.host.url=http://169.56.94.238:9000 \
                          -Dsonar.login=ab7b7953ba9bfdd8c93dc960c4988be84e6dd314"
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh "mvn package -DskipTests=true"
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
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
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

