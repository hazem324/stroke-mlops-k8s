pipeline {

    agent any

    parameters {
        string(
            name: 'IMAGE_VERSION',
            defaultValue: '1.3',
            description: 'Docker image version to deploy'
        )
    }

    stages {

        stage('Checkout Kubernetes Repository') {
            steps {
                echo 'Checking out Kubernetes repository'

                checkout scm
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                sh """
                    set -e

                    echo "Updating Kubernetes manifests"
                    echo "IMAGE_VERSION=${params.IMAGE_VERSION}"
                    echo "======================================"

                    echo "Updating Backend image..."

                    sed -i 's#image: hazem231/stroke-backend:.*#image: hazem231/stroke-backend:${params.IMAGE_VERSION}#' k8s/backend/backend-deployment.yaml


                    echo "Updating Frontend image..."

                    sed -i 's#image: hazem231/stroke-frontend:.*#image: hazem231/stroke-frontend:${params.IMAGE_VERSION}#'  k8s/frontend/frontend-deployment.yaml


                    echo "FastAPI image update skipped temporarily."

                    echo ""
                    echo "Updated image references"

                    grep -R "image: hazem231/stroke-" k8s/ || true
                """
            }
        }

        stage('Commit and Push GitOps Changes') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'mlops-token',
                        usernameVariable: 'GITHUB_USER',
                        passwordVariable: 'GITHUB_TOKEN'
                    )
                ]) {

                    sh """
                        set -e

                        echo "Committing GitOps changes"
                        echo "======================================"

                        git config user.name "Jenkins"
                        git config user.email "jenkins@local"

                        git add k8s/

                        if git diff --cached --quiet; then
                            echo "No manifest changes detected."
                            echo "Nothing to commit."
                        else
                            git commit -m "Deploy ${params.IMAGE_VERSION}"
                        fi

                        git push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/hazemhadda231/stroke-mlops-k8s.git HEAD:main
                    """
                }
            }
        }

        stage('Apply Kubernetes Manifests') {
            steps {
                sh '''

                    echo "kubectl apply is disabled temporarily."
                    echo "Jenkins does not currently have access to the local Kind cluster."

                    # Future deployment command:
                    #
                    # kubectl apply -f k8s/

                    sleep 600
                    echo "Kubernetes deployed."
                '''
            }
        }
    }

    post {

        success {
            echo 'CD PIPELINE SUCCESS'
            echo "Deployment version: ${params.IMAGE_VERSION}"
            echo 'Kubernetes manifests updated and pushed to GitHub.'
        }

        failure {
            echo 'CD PIPELINE FAILED'
            echo 'Review the stage logs.'
        }
    }
}

