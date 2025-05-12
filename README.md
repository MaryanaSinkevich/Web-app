# React Application with DevOps Pipeline

This project is a React application with a complete DevOps pipeline setup that includes Docker containerization and Jenkins CI/CD configuration.

## Development Setup

### Prerequisites

- Node.js 18 or later
- npm or yarn
- Docker
- Jenkins (optional, for CI/CD)

### Local Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

## Docker Setup

### Building the Docker Image

```bash
# Build the Docker image
docker build -t react-app:latest .

# Run the container
docker run -p 8080:80 react-app:latest
```

### Using Docker Compose

```bash
# Start the development environment
docker-compose up dev

# Start the staging environment
docker-compose up staging

# Start the production environment
docker-compose up prod
```

## CI/CD Pipeline

This project includes a Jenkins pipeline configuration (`Jenkinsfile`) that:

1. Checks out the code from the repository
2. Installs dependencies
3. Runs linting and tests
4. Builds the application
5. Creates a Docker image
6. Deploys the application to the specified environment

### Jenkins Setup

1. Create a new Pipeline job in Jenkins
2. Configure the job to use "Pipeline script from SCM"
3. Specify your repository URL
4. Set the script path to `Jenkinsfile`
5. Save and run the pipeline

### Pipeline Parameters

- `DOCKER_TAG`: Tag for the Docker image (default: "latest")
- `ENVIRONMENT`: Deployment environment (choices: "dev", "staging", "prod")
- `RUN_TESTS`: Whether to run tests during the build (default: true)

## Accessing the Application

After deployment, the application will be available at:

- Development: `http://localhost:8081`
- Staging: `http://localhost:8082`
- Production: `http://localhost:8083`