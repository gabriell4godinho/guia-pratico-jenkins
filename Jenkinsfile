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
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    powershell '(Get-Content ./k8s/deployment.yaml) -replace "{{tag}}", "${env.BUILD_ID}" | Set-Content ./k8s/deployment.yaml'
                    bat 'kubectl apply --validate=false -f k8s/deployment.yaml'
                    bat 'kubectl cluster-info dump --output-directory=cluster-info'
                }
            }
        } 
    }
}
