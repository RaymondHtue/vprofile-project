pipeline {
    agent {
        node {
            label 'buildin-node'
        }
    }
    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK11'
    }
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Mgr8ukwxaK+/iEI='
        CENTRAL_REPO = 'vpro-maven-central'
        RELEASE_REPO = 'vprofile-release'
        NEXUSIP = '192.168.1.133'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = '9f56f9e2-6e77-4b0c-8324-8c214db7af78'
        NEXUS_URL = 'nexus.visibleone.app'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Build Success'
                    echo 'Now Archiving the Artifacts'
                    archiveArtifacts artifacts: '**/*.war'
                }
                failure {
                    echo 'Build Failed'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml  checkstyle:checkstyle'

            }
        }
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonarscanner'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                // Add to trigger
                waitForQualityGate abortPipeline: true
             }
        }
        }
        stage('UploadArtifact') {
            steps {
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'https':,
                nexusUrl: "${NEXUS_URL}",
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RESLEASE_REPO}",
                credentialsId: "${NEXUS_LOGIN}",
                artifacts: [
                    [artifactId: 'vprofileapp',
                    classifier: '',
                    file: 'target/vprofile-v2.war',
                    type: 'war']
                 ]
                )
            }
        }
    }
}
