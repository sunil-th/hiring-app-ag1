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
        stage('Checkout K8s Manifests') {
            steps {
                git branch: 'main', url: "${DEPLOY_REPO}"
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    echo "Updating image tag in deployment.yaml to ${params.IMAGE_TAG}"
                    sh '''
                        sed -i "s|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g" dev/deployment.yaml
                        cat dev/deployment.yaml
                    '''
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh '''
                        git config user.name "jenkins"
                        git config user.email "jenkins@ci.local"
                        git add .
                        git commit -m "Updated deployment.yaml with new image tag ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/betawins/Hiring-app-argocd.git main
                    '''
                }
            }
        }

        stage('Trigger ArgoCD Sync (Optional)') {
            steps {
                echo "ArgoCD will automatically detect the Git change and sync the application."
            }
        }
    }
}
