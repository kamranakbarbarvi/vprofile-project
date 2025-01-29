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
      }
} 