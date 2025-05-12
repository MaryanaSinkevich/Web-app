pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Tag for Docker image')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests during build')
    }

    environment {
        NODE_VERSION = '18'
        DOCKER_IMAGE = 'react-app'
    }

    stages {
        stage('Setup Node.js') {
            steps {
                script {
                    // Use nvm or node tool to ensure correct Node.js version
                    sh """
                        export NVM_DIR="\$HOME/.nvm"
                        [ -s "\$NVM_DIR/nvm.sh" ] && . "\$NVM_DIR/nvm.sh"
                        nvm install ${NODE_VERSION}
                        nvm use ${NODE_VERSION}
                    """
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Type Check') {
            steps {
                sh 'npx tsc --noEmit'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${DOCKER_IMAGE}:${params.DOCKER_TAG}-${params.ENVIRONMENT}"
                    def envTag = "${DOCKER_IMAGE}:${params.ENVIRONMENT}"
                    
                    // Build the Docker image
                    sh "docker build -t ${imageTag} ."
                    sh "docker tag ${imageTag} ${envTag}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def port
                    switch(params.ENVIRONMENT) {
                        case 'dev':
                            port = 8081
                            break
                        case 'staging':
                            port = 8082
                            break
                        case 'prod':
                            port = 8083
                            break
                        default:
                            error "Invalid environment specified"
                    }
                    
                    def containerName = "react-app-${params.ENVIRONMENT}"
                    
                    // Stop and remove existing container if it exists
                    sh "docker stop ${containerName} || true"
                    sh "docker rm ${containerName} || true"
                    
                    // Run the new container
                    sh """
                        docker run -d \
                            -p ${port}:80 \
                            --name ${containerName} \
                            --restart unless-stopped \
                            ${DOCKER_IMAGE}:${params.ENVIRONMENT}
                    """
                    
                    echo "Application deployed to http://localhost:${port}"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! The application is now available."
        }
        failure {
            echo "Pipeline failed. Please check the logs for more information."
        }
        always {
            // Clean workspace
            cleanWs()
            
            // Clean up old Docker images to prevent disk space issues
            script {
                try {
                    sh "docker image prune -f"
                } catch (err) {
                    echo "Warning: Failed to clean up Docker images: ${err}"
                }
            }
        }
    }
}
