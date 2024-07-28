# Continuation Exercise: Extend the Declarative Pipeline

## Objective

Extend the existing Jenkins declarative pipeline by adding two new functions:
1. Read the `package.json` file to set the build name.
2. Create a report summarizing all stages and save it as a PDF file, attaching it to the Jenkins build GUI.

## Prerequisites

- Completion of the previous exercise.

## Instructions

### Step 1: Create Directory Structure

Create the following directory structure in your Jenkins shared library repository:

```
.
├── src/
│ └── org/
│ └── jenkins/
│ └── mypipeline/
│ ├── BuildDockerImage.groovy
│ ├── CloneRepository.groovy
│ ├── PublishDockerImage.groovy
│ ├── TestDockerImage.groovy
│ ├── ReadPackageJson.groovy
│ └── CreateReport.groovy
└── vars/
└── myPipeline.groovy
```


### Step 2: Create Functions for Each Stage

#### `CloneRepository.groovy`

```groovy
package org.jenkins.mypipeline

def call(String repoUrl, String branch) {
    git url: repoUrl, branch: branch
}
```

#### `BuildDockerImage.groovy`

```groovy
package org.jenkins.mypipeline

def call(String imageName) {
    docker.build(imageName, '-f Dockerfile .')
}
```

#### `TestDockerImage.groovy`

```groovy
package org.jenkins.mypipeline

def call(String imageName) {
    docker.image(imageName).inside {
        sh 'python -m unittest discover -s tests'
    }
}
```

#### `PublishDockerImage.groovy`

```groovy
package org.jenkins.mypipeline

def call(String imageName, String repo, String artifactoryUrl, String credentialsId, String buildNumber) {
    def server = Artifactory.server(credentialsId)
    def dockerBuildInfo = Artifactory.dockerBuild(
        server: server,
        tool: 'docker',
        image: "${repo}/${imageName}",
        dockerRegistryUrl: artifactoryUrl
    )

    dockerBuildInfo.buildInfo.name = "${repo}/${imageName}"
    dockerBuildInfo.buildInfo.number = buildNumber

    docker.withRegistry(artifactoryUrl, credentialsId) {
        docker.image(imageName).push('latest')
    }

    server.publishBuildInfo(dockerBuildInfo.buildInfo)
}
```

#### `ReadPackageJson.groovy`

```groovy
package org.jenkins.mypipeline

import groovy.json.JsonSlurper

def call(String filePath) {
    def packageJson = new File(filePath).text
    def json = new JsonSlurper().parseText(packageJson)
    def name = json.name
    def version = json.version
    return "${name}.${version}"
}
```

#### `CreateReport.groovy`

```groovy
package org.jenkins.mypipeline

def call(String reportFilePath, Map stageStatus) {
    def reportContent = "Build Report\n\n"
    stageStatus.each { stage, status ->
        reportContent += "${stage}: ${status}\n"
    }

    def reportFile = new File(reportFilePath)
    reportFile.text = reportContent

    // Convert the text report to PDF
    sh "echo \"${reportContent}\" | enscript -B -o - | ps2pdf - ${reportFilePath}.pdf"
}
```

### Step 3: Define the Declarative Pipeline

#### `myPipeline.groovy`

```groovy
def call(Map config) {
    pipeline {
        agent any

        environment {
            DOCKER_IMAGE = config.dockerImage
            ARTIFACTORY_URL = config.artifactoryUrl
            ARTIFACTORY_REPO = config.artifactoryRepo
            ARTIFACTORY_CREDENTIALS = config.artifactoryCredentials
        }

        stages {
            stage('Clone Repository') {
                steps {
                    script {
                        org.jenkins.mypipeline.CloneRepository(config.repoUrl, config.branch)
                    }
                }
            }

            stage('Set Build Name') {
                steps {
                    script {
                        def buildName = org.jenkins.mypipeline.ReadPackageJson('package.json')
                        currentBuild.displayName = "${buildName}#${env.BUILD_NUMBER}"
                    }
                }
            }

            stage('Build Docker Image') {
                steps {
                    script {
                        org.jenkins.mypipeline.BuildDockerImage(DOCKER_IMAGE)
                    }
                }
            }

            stage('Test Docker Image') {
                steps {
                    script {
                        org.jenkins.mypipeline.TestDockerImage(DOCKER_IMAGE)
                    }
                }
            }

            stage('Publish Docker Image') {
                steps {
                    script {
                        org.jenkins.mypipeline.PublishDockerImage(DOCKER_IMAGE, ARTIFACTORY_REPO, ARTIFACTORY_URL, ARTIFACTORY_CREDENTIALS, env.BUILD_NUMBER)
                    }
                }
            }

            stage('Create Report') {
                steps {
                    script {
                        def stageStatus = [
                            'Clone Repository': 'PASS',
                            'Set Build Name': 'PASS',
                            'Build Docker Image': 'PASS',
                            'Test Docker Image': 'PASS',
                            'Publish Docker Image': 'PASS'
                        ]
                        org.jenkins.mypipeline.CreateReport('build_report', stageStatus)
                        archiveArtifacts artifacts: 'build_report.pdf', allowEmptyArchive: true
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
}
```

### Step 4: Configure the Jenkins Job to Use the Shared Library
1. Open your Jenkins job configuration.
2. Under "Pipeline", select "Pipeline script from SCM".
3. Configure your SCM repository and branch.
4. Add your shared library in the "Pipeline Libraries" section:
    * Name: my-shared-library
    * Default version: master (or any branch/tag you want to use)
    * Load implicitly: Checked (if you want it always loaded)
    * Allow overriding default version: Checked

### Step 5: Define Pipeline Configuration in Jenkinsfile
Create or update the Jenkinsfile in your repository with the following content:

```groovy
@Library('my-shared-library') _

myPipeline([
    dockerImage: 'my_calculator_project:latest',
    repoUrl: 'https://your-git-repo-url.git',
    branch: 'main',
    artifactoryUrl: 'https://your-artifactory-instance/artifactory',
    artifactoryRepo: 'your-docker-repo',
    artifactoryCredentials: 'artifactory-credentials-id'
])
```

By following this exercise, you will have extended your Jenkins declarative pipeline to include reading the package.json file for setting the build name and creating a summary report of all stages, saving it as a PDF, and attaching it to the Jenkins build GUI.