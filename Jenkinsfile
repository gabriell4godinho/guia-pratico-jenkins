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
                    script {
                        // Substituição segura no Windows
                        def tag = env.BUILD_ID
                        def deploymentFile = 'k8s/deployment.yaml'

                        bat "powershell -Command \"(Get-Content ${deploymentFile}) -replace '\\{\\{tag\\}\\}', '${tag}' | Set-Content ${deploymentFile}\""

                        bat 'kubectl cluster-info'

                        bat 'kubectl apply --validate=false -f k8s/deployment.yaml'

                        bat 'kubectl cluster-info dump --output-directory=cluster-info'
                    }
                }
            }
        } 
    }
}

