pipeline {
    agent {
        label 'docker'
    }
    environment {
        imageName = "petclinic"
        registryCredentials = "nexus"
        registry = "20.62.162.174:49153/repository/docker-private-repo/"
        dockerImage = ''
    }
    stages {
        stage('Building our image') {
            steps {
                script {
                    dockerImage = docker.build imageName
                }
            }
        }
        stage('Uploading to Nexus') {
            steps {
                script {
                    docker.withRegistry( 'http://'+registry, registryCredentials ) {
                    dockerImage.push('latest')
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
               sh '''
               docker run -d -p 8080:8080 --rm --name demoapp ' + registry + imageName
               '''
            }
        }
    }
}
