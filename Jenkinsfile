pipeline {
    agent any

    environment {
        REGISTRY = "myregistry123ali.azurecr.io"
        IMAGE_NAME = "myapp"
        ACR_CREDENTIALS = credentials('acr-service-principal')
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
                    docker.withRegistry("https://${REGISTRY}", 'acr-service-principal') {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                sh """
                sed -i 's|repository:.*|repository: ${REGISTRY}/${IMAGE_NAME}|' charts/values.yaml
                sed -i 's|tag:.*|tag: "${BUILD_NUMBER}"|' charts/myapp/values.yaml
                git config --global user.email "krishnaalikha236@gmail.com"
                git config --global user.name "Alikha17"
                git commit -am "Update image tag to ${BUILD_NUMBER}"
                git push origin main
                """
            }
        }
    }
}

