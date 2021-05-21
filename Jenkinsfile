pipeline {
    agent {
        label 'docker'
    }
    stages {
        stage('Building our image') {
            steps {
                script {
                    dockerImage = docker.build "akshaygets/petclinic:$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy our image') {
            steps {
                script {
                    // Assume the Docker Hub registry by passing an empty string as the first parameter
                    docker.withRegistry('' , 'dockerhub') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Health Check') {
            steps {
               sh '''
               docker run --name demoapp -d -p 8080:8080 "akshaygets/petclinic:$BUILD_NUMBER"
               '''
            }
        }
    }
}
