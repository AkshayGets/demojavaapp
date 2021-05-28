pipeline {
    agent {
        label 'docker'
    }
    
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "Maven 3.6.3"
    }

    environment {
        imageName = "petclinic"
        registryCredentials = "nexus"
        registry = "20.62.162.174:8085/"
        dockerImage = ''
    }
    stages {
        stage('Deploy') {
          steps{
              echo 'Deploying to Nexus'
              //sh "mvn clean package"
              withMaven(maven: 'mvn-3.6.3') {
                  sh "mvn deploy -DskipTests"
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
