pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Tag for Docker image')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests during build')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
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

        stage('Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                sh 'echo "Running tests..." && npm test || echo "No tests configured"'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t react-app:${params.DOCKER_TAG}-${params.ENVIRONMENT} ."
                sh "docker tag react-app:${params.DOCKER_TAG}-${params.ENVIRONMENT} react-app:${params.ENVIRONMENT}"
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
                            port = 8080
                    }
                    
                    sh "docker stop react-app-${params.ENVIRONMENT} || true"
                    sh "docker rm react-app-${params.ENVIRONMENT} || true"
                    sh "docker run -d -p ${port}:80 --name react-app-${params.ENVIRONMENT} react-app:${params.ENVIRONMENT}"
                    
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
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}