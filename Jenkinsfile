pipeline {

    agent any

    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK8'
    }
    environment{
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
		NEXUS_PASS = 'kamran@2024'
		RELEASE_REPO = 'vprofile-release'
		CENTRAL_REPO = 'vpro-maven-central'
		NEXUSIP = '10.164.0.133'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
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
                    echo 'now archiving'
                    archiveArtifacts artifacts: '**/*.war'
                }
                failure {
                    echo 'Build Failed'
                }
            }
        }

        stage ('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        
        }
        stage ('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }

            }

         stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
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
                    waitForQualityGate abortPipeline: true
                }
            }
         }

        stage ("UploadArtifact") {

        steps {
        nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        groupId: 'QA',
        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        repository: "${RELEASE_REPO}",
        credentialsId: "${NEXUS_LOGIN}",
        artifacts: [
            [artifactId: 'vproapp',
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'war']
           ]
           )   
        }

        }

    }  

   post {
        always {
            script {
                def status = currentBuild.currentResult
                def message = [
                    "text": "*Jenkins Pipeline Notification*\n" +
                             "Project: *${env.JOB_NAME}*\n" +
                             "Build Number: *${env.BUILD_NUMBER}*\n" +
                             "Status: *${status}*\n" +
                             "View Build: ${env.BUILD_URL}"
                ]
                def jsonMessage = new groovy.json.JsonBuilder(message).toString()
                
                httpRequest(
                    httpMode: 'POST',
                    acceptType: 'APPLICATION_JSON',
                    contentType: 'APPLICATION_JSON',
                    requestBody: jsonMessage,
                    url: 'https://chat.googleapis.com/v1/spaces/AAAABO-z3SQ/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=QuesUHehRtMpF6rmclbHadnTUN5lBuOdEMO61UmMEZY'
                )
            }
        }
    }



} 