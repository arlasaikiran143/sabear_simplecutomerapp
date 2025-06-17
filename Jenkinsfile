pipeline {
    agent any
    tools {
        maven "MVN_HOME"
    }
    environment {
        // Nexus settings
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.95.29.162:8081"
        NEXUS_REPOSITORY = "SimpleCustomerApp"
        NEXUS_CREDENTIAL_ID = "nexus_keygen"

        // Tomcat deployment settings
        TOMCAT_URL = "http://13.221.159.29:8081/manager/text/deploy?path=/SimpleCustomerApp&update=true"
        TOMCAT_CREDENTIAL_ID = "tomcat"
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    git 'https://github.com/arlasaikiran143/sabear_simplecutomerapp.git'
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
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path
                    def artifactExists = fileExists(artifactPath)

                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }

        stage("deploy to tomcat") {
            steps {
                script {
                    def warFile = "target/SimpleCustomerApp.war"
                    withCredentials([usernamePassword(credentialsId: TOMCAT_CREDENTIAL_ID, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS -T ${warFile} "${TOMCAT_URL}"
                        """
                    }
                }
            }
        }
    }
}
