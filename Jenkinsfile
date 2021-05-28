pipeline {
    agent {
        label 'docker'
    }
    
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "Maven 3.6.3"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        imageName = "petclinic"
        registryCredentials = "nexus"
        registry = "20.62.162.174:8085/"
        dockerImage = ''
    }
    stages {
        stage('Deploy') {
          steps{
                script {
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
                    filesByGlob = findFiles(glob: "/app/target/*.${pom.packaging}");
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
                            nexusUrl: registry,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: nexus,
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
        stage('Building our image') {
            steps {
                script {
                    dockerImage = docker.build "petclinic:$BUILD_NUMBER"
                }
            }
        }
        stage('Uploading to Nexus') {
            steps {
                script {
                    docker.withRegistry( 'http://'+registry, registryCredentials ) {
                    dockerImage.push()
                }
            }
        }
        }
        stage('stop previous containers') {
            steps {
                sh 'docker ps -f name=demoapp -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -fname=demoapp -q | xargs -r docker container rm'
            }
        }
        stage('Deploy to Local') {
            steps {
               sh 'docker run -d -p 8080:8080 --rm --name demoapp ' + registry + "petclinic:$BUILD_NUMBER"
            }
        }
    }
}
