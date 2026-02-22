pipeline {
    agent any 
    tools {
        maven "MVN_HOME"
    }
    environment {
        // Nexus configuration
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.133.145.136:8081"
        NEXUS_REPOSITORY = "devops"
        NEXUS_CREDENTIAL_ID = "Nexus_server"

        // Slack configuration
        SLACK_CHANNEL = "#jenkins-integration" // replace with your Slack channel
        SLACK_CREDENTIAL_ID = "slack-token-id"   // Jenkins credential id for Slack token
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, color: '#FFFF00', message: "Starting build for job ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                    git 'https://github.com/betawins/spring3-mvc-maven-xml-hello-world-1.git'
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
                }
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version $BUILD_NUMBER"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: "${BUILD_NUMBER}",
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                        slackSend(channel: SLACK_CHANNEL, color: 'good', message: "Build #${env.BUILD_NUMBER} SUCCESS for ${env.JOB_NAME}. Artifact uploaded to Nexus.")
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
    post {
        failure {
            script {
                slackSend(channel: SLACK_CHANNEL, color: 'danger', message: "Build #${env.BUILD_NUMBER} FAILED for ${env.JOB_NAME}. Check Jenkins console for details.")
            }
        }
    }
}
