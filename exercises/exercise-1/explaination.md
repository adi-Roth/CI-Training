# Explanation of the Pipeline Script
## Environment Variables:

- **DOCKER_IMAGE:** Name and tag of the Docker image.
- **ARTIFACTORY_URL:** URL of the Artifactory instance.
- **ARTIFACTORY_REPO:** Name of the Artifactory repository for Docker images.
- **ARTIFACTORY_CREDENTIALS:** Credentials ID for Artifactory (configure in Jenkins credentials).

## Stages:

- **Clone Repository:** Clones the Git repository.
- **Build Docker Image:** Builds the Docker image using the Dockerfile.
- **Test Docker Image:** Tests the Docker image by running unit tests inside the container.
- **Publish Docker Image:** Pushes the Docker image to the Artifactory repository and publishes build information.

## Post Actions:

- **always:** Cleans up the workspace after the build.

## Configuring Jenkins Credentials

**Go to:** Jenkins Dashboard -> Manage Jenkins -> Manage Credentials.

### Add a new credential with the following details:
- **Kind:** Username with password
- **Username:** Artifactory username
- **Password:** Artifactory password
- **ID:** artifactory-credentials-id (match this with ARTIFACTORY_CREDENTIALS in the script).

## Final Steps
- **Save the Pipeline Job:** Save the Jenkins pipeline job configuration.
- **Run the Pipeline Job:** Trigger the pipeline job manually or configure it to run on a schedule or based on repository changes.

By following this exercise, you will automate the process of testing, building, and publishing your Docker image to an Artifactory repository using a Jenkins scripted pipeline.