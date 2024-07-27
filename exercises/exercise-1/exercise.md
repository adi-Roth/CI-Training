# Exercise: Jenkins Scripted Pipeline for Docker Project

## Objective

Create a Jenkins scripted pipeline to automate the process of testing, building, and publishing a Docker image (production stage) to a custom Artifactory repository.

## Prerequisites

1. **Jenkins Server**: Ensure you have access to a Jenkins server.
2. **Jenkins Plugins**: Make sure the following plugins are installed:
   - Docker Pipeline Plugin
   - Artifactory Plugin
3. **Artifactory Repository**: Have an Artifactory repository available for Docker images.
4. **Docker**: Ensure Docker is installed on your Jenkins agent.
5. **Git**: Ensure Git is installed on your Jenkins agent and the repository is accessible.

## Instructions

### Step 1: Create a New Jenkins Pipeline Job

1. Open your Jenkins dashboard.
2. Click on "New Item".
3. Enter a name for your job (e.g., `my-calculator-docker-pipeline`).
4. Select "Pipeline" and click "OK".

### Step 2: Configure the Pipeline Script

1. In the Pipeline section, select "Pipeline script" and enter the following scripted pipeline code.

### Pipeline Script

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'my_calculator_project:latest'
        ARTIFACTORY_URL = 'https://your-artifactory-instance/artifactory'
        ARTIFACTORY_REPO = 'your-docker-repo'
        ARTIFACTORY_CREDENTIALS = 'artifactory-credentials-id'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://your-git-repo-url.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE, '-f Dockerfile .')
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    // Test the Docker image (assuming tests are run in the build stage of Dockerfile)
                    docker.image(DOCKER_IMAGE).inside {
                        sh 'python -m unittest discover -s tests'
                    }
                }
            }
        }

        stage('Publish Docker Image') {
            steps {
                script {
                    def server = Artifactory.server(ARTIFACTORY_CREDENTIALS)
                    def dockerBuildInfo = Artifactory.dockerBuild(
                        server: server,
                        tool: 'docker',
                        image: "${ARTIFACTORY_REPO}/${DOCKER_IMAGE}",
                        dockerRegistryUrl: ARTIFACTORY_URL
                    )

                    dockerBuildInfo.buildInfo.name = "${ARTIFACTORY_REPO}/${DOCKER_IMAGE}"
                    dockerBuildInfo.buildInfo.number = "${env.BUILD_NUMBER}"

                    docker.withRegistry(ARTIFACTORY_URL, ARTIFACTORY_CREDENTIALS) {
                        docker.image(DOCKER_IMAGE).push('latest')
                    }

                    server.publishBuildInfo(dockerBuildInfo.buildInfo)
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```