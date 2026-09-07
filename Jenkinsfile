pipeline {
    agent any

   environment {
        // Match the exact image string in your YAML
        DOCKER_IMAGE    = 'docker.io/shgupta140/trendify-app'
        IMAGE_TAG       = "v${env.BUILD_NUMBER}"
        DOCKER_CREDS_ID = 'shgupta140-Docker-Cred'
        
        // Point to the repository holding 01-deployment.yml
        GITOPS_REPO     = 'github.com/shgupta140/Trendify-App.git' 
        GITOPS_BRANCH   = 'main'
        GIT_CREDS_ID    = 'shgupta140-max-git-repo-cred'
        
        // The exact file name
        TARGET_YAML     = 'kubedefs/01-deployment.yml' 
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building image ${DOCKER_IMAGE}:${IMAGE_TAG}..."
                    // Build with the specific version tag and the latest tag
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDS_ID, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    // Authenticate and push both tags
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {
                withCredentials([usernamePassword(credentialsId: GIT_CREDS_ID, passwordVariable: 'GIT_TOKEN', usernameVariable: 'GIT_USER')]) {
                    sh """
                        # Clone the GitOps repo into a temporary workspace
                        git clone https://${GIT_USER}:${GIT_TOKEN}@${GITOPS_REPO} gitops-workspace
                        cd gitops-workspace
                        
                        # Configure Git identity for the audit trail
                        git config user.name "Shashank Gupta"
                        git config user.email "shgupta140@gmail.com"
                        
                        # Use sed to find the old image tag and replace it with the new build tag
                        sed -i "s|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|g" ${TARGET_YAML}
                        
                        # Commit and push the manifest update back to the GitOps repo
                        git add ${TARGET_YAML}
                        git commit -m "Auto-deploy: Update App image to ${IMAGE_TAG}"
                        git push origin ${GITOPS_BRANCH}
                    """
                }
            }
        }
    }
    
    post {
        always {
            // Clean up credentials and temporary files to secure the Jenkins node
            sh "docker logout || true"
            sh "rm -rf gitops-workspace || true"
        }
    }
}