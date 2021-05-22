pipeline {
    agent {
        label 'docker'
    }
    environment {
        imageName = "petclinic"
        registryCredentials = "nexus"
        registry = "20.62.162.174:8085/"
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

        stage('Deploy to Local') {
            steps {
               sh '''
               docker run -d -p 8080:8080 --rm --name demoapp ' + registry + imageName
               '''
            }
        }
    }
}
