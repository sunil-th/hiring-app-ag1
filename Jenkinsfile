pipeline {
    agent any

    environment {
        DEPLOY_REPO = "https://github.com/sunil-th/Hiring-app-argocd.git"
        IMAGE_NAME = "sunildev99/argocd"
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker image tag to deploy')
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.IMAGE_TAG?.trim()) {
                        error("IMAGE_TAG parameter is required! Please pass a valid Docker image tag.")
                    }
                }
            }
        }

        stage('Checkout K8s Manifests') {
            steps {
                git branch: 'main', url: "${DEPLOY_REPO}"
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    echo "Updating image tag in deployment.yaml to ${params.IMAGE_TAG}"
                    sh """
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${params.IMAGE_TAG}|g' dev/deployment.yaml
                        echo "----- Updated Deployment YAML -----"
                        cat dev/deployment.yaml
                    """
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh """
                        git config user.name "jenkins"
                        git config user.email "jenkins@ci.local"
                        git add .
                        git commit -m "Updated deployment.yaml with new image tag ${params.IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/sunil-th/Hiring-app-argocd.git main
                    """
                }
            }
        }

        stage('Trigger ArgoCD Sync (Optional)') {
            steps {
                echo "✅ ArgoCD will automatically detect the Git change and sync the deployment."
            }
        }
    }
}
