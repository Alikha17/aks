pipeline {
    agent any

    environment {
        REGISTRY = "myregistry123ali.azurecr.io"
        IMAGE_NAME = "myapp"
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

        stage('Update Helm Values & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-pat',
                                 passwordVariable: 'GIT_TOKEN',
                                 usernameVariable: 'GIT_USER')]) {
                    sh '''
                        # 1. Configure Git
                        git config user.email "krishnaalikha236@gmail.com"
                        git config user.name "Alikha17"

                        # 2. Ensure we are on main branch
                        git checkout main

                        # 3. Update Helm values
                        sed -i "s|repository:.*|repository: ${REGISTRY}/${IMAGE_NAME}|" charts/values.yaml
                        sed -i "s|tag:.*
