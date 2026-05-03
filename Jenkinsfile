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
                checkout([$class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Alikha17/aks.git',
                        credentialsId: 'github-pat'
                    ]]
                ])
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
                sed -i 's|tag:.*|tag: "${BUILD_NUMBER}"|' charts/values.yaml

                git config user.email "krishnaalikha236@gmail.com"
                git config user.name "Alikha17"

                git add charts/values.yaml
                git commit -m "Update image tag to ${BUILD_NUMBER}"
                git push origin main
                """
            }
        }
    }
}
