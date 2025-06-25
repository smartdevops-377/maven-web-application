pipeline {
    agent any

    tools {
        maven 'Maven3'    // Make sure this matches the name in Jenkins Tools config
        jdk 'Java17'      // Use the name of your configured JDK
    }

    environment {
        SONARQUBE_ENV = 'SonarQubeServer'      // Name of Sonar server in Jenkins config
        NEXUS_URL = 'http://34.173.82.220:8081/repository/maven-snapshots/'
        TOMCAT_URL = 'http://34.55.221.192:8080/manager/text'
        WAR_FILE = 'target/SampleWebApplication.war'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Navya89k/maven-web-application.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload to Nexus') {
    steps {
        nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: '34.173.82.220:8081',             
            groupId: 'com.javacodegeeks',
            version: '1.0-SNAPSHOT',
            repository: 'maven-snapshots',           
            credentialsId: 'nexus-creds-id',
            artifacts: [
                [artifactId: 'maven-web-app',
                 classifier: '',
                 file: 'target/SampleWebApplication.war',
                 type: 'war']
            ]
        )
    }
}


        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-cred-id', 
                                          url: "${TOMCAT_URL}")],
                       contextPath: 'SampleWebApplication',
                       war: "${env.WAR_FILE}"
                echo " deployment successful"
            }
        }
    }
}

