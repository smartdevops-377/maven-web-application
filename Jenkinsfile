/*  Jenkinsfile  */
pipeline {
    /* Don’t run anything on the controller unless a stage asks for it. */
    agent none                       // :contentReference[oaicite:1]{index=1}

    options {
        /* Keep builds orderly */
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        SONARQUBE_ENV = 'SonarQubeServer'      // Name of Sonar server in Jenkins config
        NEXUS_URL = 'http://34.173.82.220:8081/repository/maven-snapshots/'
        TOMCAT_URL = 'http://34.55.221.192:8080/manager/text'
        WAR_FILE = 'target/SampleWebApplication.war'
        MAVEN_OPTS = '-Dmaven.repo.local=$HOME/.m2/repository'
    }

    /* ─────────────────────  PIPELINE STAGES  ───────────────────── */
    stages {

        /* 0.  Quick pre-flight on the controller (“master”)           */
        stage('Controller prep') {
            /* Runs on the built-in node.  Label is literally ‘master’. */
            agent { label 'main' }
            steps {
                echo "Job #${env.BUILD_NUMBER} kicked off by ${currentBuild.getBuildCauses()[0].shortDescription}"
            }
        }

        /* 1.  Clone + Build + Unit tests on the Linux build agent     */
        stage('Build & Unit Test') {
            agent { label 'slavenode' }          // the agent you created earlier
            steps {
                checkout scm                 // Git checkout
                sh 'mvn -B clean verify'     // compile + unit tests
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        /* 2.  Static analysis (SonarQube) on the same build agent     */
        stage('Sonar Scan') {
            agent { label 'slavenode' }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        /* 3.  Publish the WAR to Nexus                                */
        stage('Publish Artifact') {
            agent { label 'slavenode' }
            steps {
                nexusArtifactUploader(
                    nexusVersion:  'nexus3',
                    nexusUrl:      'http://34.173.82.220:8081',
                    protocol:      'http',
                    groupId:       'com.example',
                    version:       "${env.BUILD_NUMBER}",
                    repository:    'maven-releases',
                    credentialsId: 'nexus-creds',
                    artifacts: [[artifactId: 'myapp',
                                 type: 'war',
                                 file: 'target/myapp.war']]
                )
            }
        }

        /* 4.  Deploy to Tomcat (optional separate “deploy” agent)     */
        stage('Deploy to Tomcat') {
            /* Change to label 'deploy' if you made a dedicated node.  */
            agent { label 'master' }
            when { branch 'main' }
            steps {
                sh '''
                  scp -o StrictHostKeyChecking=no target/myapp.war jenkins@tomcat-host:/opt/tomcat/webapps/
                  ssh jenkins@tomcat-host "/opt/tomcat/bin/catalina.sh stop && sleep 5 && /opt/tomcat/bin/catalina.sh start"
                '''
            }
        }

        /* 5.  If anything failed, run a “Stage Z” on the controller   */
        stage('Failure cleanup') {
            agent { label 'master' }
            /* Run only when previous stages fail */
            when { expression { currentBuild.currentResult == 'FAILURE' } }
            steps {
                echo 'Previous stage failed – performing rollback / alerts'
                // Put your rollback logic here
            }
        }
    }

    /* ──────────────────────────  GLOBAL POST  ───────────────────── */
    post {
        success {
            echo '🎉  Pipeline completed successfully'
        }
        unstable {
            echo '⚠️  Some tests failed'
        }
        failure {
            mail to:      'dev-team@example.com',
                 subject: "❌  Build #${env.BUILD_NUMBER} failed",
                 body:    "Console: ${env.BUILD_URL}"
        }
    }
}

