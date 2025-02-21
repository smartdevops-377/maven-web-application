node {
    def mavenapp = tool name : "mvn3.9.9"
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '2')), pipelineTriggers([cron('* * * * *')])])
    stage("checkout") {
        git branch: 'main', credentialsId: '5b3c31cc-0bb2-446c-a991-7681eb94b2d8', url: 'https://github.com/smartdevops-377/maven-web-application.git'
    }
    
    stage('build') {
      
    sh "${mavenapp}/bin/mvn clean package"
    // some block
}
/*stage ("sonarqube")
{
    sh "${mavenapp}/bin/mvn sonar : sonar"
}
stage ("artifact upload") 
{
     sh "${mavenapp}/bin/mvn deploy"
}
*/

stage ("deploy to the container")
{
    //deploy adapters: [tomcat9(credentialsId: '1d974d15-9756-4c6f-b106-b09758995292', path: '', url: 'http://13.201.191.45:8081')], contextPath: '/', war: '**/*.war'
    echo "deployed succesfully"
}
stage ("final output")
       {
           echo " demo of multibranch pipeline"
       }
       
}

