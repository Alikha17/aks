pipeline {
    agent any

    environment {
        REGISTRY = "myregistry123ali.azurecr.io"
        IMAGE_NAME = "myapp"
        // ACR_CREDENTIALS is used by the docker steps below
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
                // Binding the 'github-pat' credential to variables
                withCredentials([usernamePassword(credentialsId: 'github-pat', 
                                 passwordVariable: 'GIT_TOKEN', 
                                 usernameVariable: 'GIT_USER')]) {
                    sh """
                        # Update the Helm values file
                        sed -i 's|repository:.*|repository: ${REGISTRY}/${IMAGE_NAME}|' charts/values.yaml
                        sed -i 's|tag:.*|tag: "${BUILD_NUMBER}"|' charts/values.yaml

                        # Git configuration
                        git config user.email "krishnaalikha236@gmail.com"
                        git config user.name "Alikha17"

                        git add charts/values.yaml
                        git commit -m "Update image tag to ${BUILD_NUMBER}"

                        # Use the credential variables in the push URL to avoid authentication prompts
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/Alikha17/aks.git main
                    """
                }
            }
        }
    }
}
