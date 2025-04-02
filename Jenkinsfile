pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gabriellagodinho/guia-pratico-jenkins:${env.BUILD_ID}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    dockerapp = docker.build(env.DOCKER_IMAGE, '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push(env.BUILD_ID)
                    }
                }
            }
        }     

        stage('Deploy no Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    bat 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    bat 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        } 
    }
}
