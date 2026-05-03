pipeline {
    agent any

    environment {
        REGISTRY = "myregistry123ali.azurecr.io"
        IMAGE_NAME = "myapp"
        ACR_CREDENTIALS = credentials('acr-service-principal') // store in Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Alikha17/aks.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to ACR') {
            steps {
                script {
                    docker.withRegistry('https://myregistry123ali.azurecr.io','acr-service-principal') {
                    docker.image("${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        
                    }
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                sh """
                sed -i 's/tag:.*/tag: "${BUILD_NUMBER}"/' charts/myapp/values.yaml
                git config --global user.email "jenkins@ci.com"
                git config --global user.name "jenkins"
                git commit -am "Update image tag to ${BUILD_NUMBER}"
                git push origin main
                """
            }
        }
    }
}

