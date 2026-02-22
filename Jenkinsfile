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
        SLACK_CHANNEL = "#jenkins-integration"
        SLACK_CREDENTIAL_ID = "slack-2"

        // Tomcat configuration
        TOMCAT_URL = "http://your-tomcat-server:8080/manager/text"
        TOMCAT_CREDENTIAL_ID = "tomcat-credentials"
        TOMCAT_CONTEXT = "/spring3-hello"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, color: '#FFFF00', message: "🔄 Starting build for job ${env.JOB_NAME} #${env.BUILD_NUMBER}", tokenCredentialId: SLACK_CREDENTIAL_ID)
                    git 'https://github.com/betawins/spring3-mvc-maven-xml-hello-world-1.git'
                }
            }
        }
        stage("mvn build") {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, color: '#FFFF00', message: "🏗️ Running Maven build...", tokenCredentialId: SLACK_CREDENTIAL_ID)
                    sh 'mvn -Dmaven.test.failure.ignore=true install'
                }
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path
                    if (fileExists artifactPath) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${BUILD_NUMBER}"
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
                        slackSend(channel: SLACK_CHANNEL, color: 'good', message: "✅ Build #${env.BUILD_NUMBER} SUCCESS. Artifact uploaded to Nexus.", tokenCredentialId: SLACK_CREDENTIAL_ID)
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
        stage("deploy to Tomcat") {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, color: '#FFFF00', message: "🚀 Deploying artifact to Tomcat ${TOMCAT_CONTEXT}", tokenCredentialId: SLACK_CREDENTIAL_ID)
                    withCredentials([usernamePassword(credentialsId: TOMCAT_CREDENTIAL_ID, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        sh """
                        curl -T ${artifactPath} "${TOMCAT_URL}${TOMCAT_CONTEXT}?update=true" --user $TOMCAT_USER:$TOMCAT_PASS
                        """
                    }
                    slackSend(channel: SLACK_CHANNEL, color: 'good', message: "🎉 Deployment to Tomcat ${TOMCAT_CONTEXT} completed successfully.", tokenCredentialId: SLACK_CREDENTIAL_ID)
                }
            }
        }
    }
    post {
        failure {
            script {
                slackSend(channel: SLACK_CHANNEL, color: 'danger', message: "❌ Build #${env.BUILD_NUMBER} FAILED for ${env.JOB_NAME}. Check Jenkins console for details.", tokenCredentialId: SLACK_CREDENTIAL_ID)
            }
        }
    }
}
