node{
    
def mavenHome = tool name: 'maven 3.9.5'

echo "Job name is: ${env.JOB_NAME}"
echo "Node name is: ${env.NODE_NAME}"
echo "Jenkins Home is: ${env.JENKINS_HOME}"
echo "Jenkins URL: ${env.JENKINS_URL}"


properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])


try{
stage('CheckOutCode'){
    sendSlackNotifications("STARTED")
git branch: 'development', credentialsId: '88df5c0f-65c7-484c-848b-00e6fe55df01', url: 'https://github.com/pushpendra369/maven-web-application.git'
}

stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}

stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn clean sonar:sonar"
}

stage('UploadArtifactIntoNexus'){
sh "${mavenHome}/bin/mvn clean deploy"
}

stage('DeployAppIntoTomcatServer'){
sshagent(['87323f6f-44a3-4c49-9022-6316560b254a']) {
   sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.41.102:/opt/apache-tomcat-9.0.83/webapps/"
}
}

}
catch(e){
currentBuild.result = "FAILURE"
    throw e
}
finally{
sendSlackNotifications(currentBuild.result)
}
}//node closing


def sendSlackNotifications(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    colorName = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    colorName = 'GREEN'
    colorCode = '#00FF00'
  } else {
    colorName = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
