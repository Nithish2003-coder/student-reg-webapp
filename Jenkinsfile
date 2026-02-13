pipeline{
    agent any
    triggers {
        githubPush()
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')
        disableConcurrentBuilds()
        timeout(10)
    }

    environment{
        SONAR_TOKEN = credentials('sonartoken')
        SONAR_HOST = "http://172.31.81.85:9000"
        tomcatSSHusername = "ec2-user"
        tomcatIP = "172.31.14.189"
    }
    tools{
        maven 'Maven-3.9.11'
    }
    stages{
        stage("Git clone"){
            steps{
                git branch: 'main', credentialsId: 'githubcredentials', url: 'https://github.com/Nithish2003-coder/student-reg-webapp.git'
            }
        }
        stage("Build stage"){
            steps{
                sh "mvn clean package"
            }
        }
        stage("sonar scan"){
            steps{
                sh "mvn clean verify sonar:sonar -Dsonar.host:${SONAR_HOST} -Dsonar.token:${SONAR_TOKEN}"
            }
        }
        stage("upload artifact to deploy"){
            steps{
                sh "mvn clean deploy"
            }
        }
        stage("Deploy war to tomcat in dev server"){
            when {
                expression { env.BRANCH_NAME ==  "development" }
            }

            steps{
                sshagent(['TomcatServer_SSH_credentials']) {
                    sh """
                       ssh -o StrictHostKeyChecking=no ${tomcatSSHusername}@${tomcatIP} sudo systemctl stop tomcat || true
                       sleep 20
                       ssh -o StrictHostKeyChecking=no ${tomcatSSHusername}@${tomcatIP} rm -f /opt/tomcat/webapps/student-reg-webapp.war
                       scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ${tomcatSSHusername}@${tomcatIP}:/opt/tomcat/webapps/student-reg-webapp.war
                       ssh -o StrictHostKeyChecking=no ${tomcatSSHusername}@${tomcatIP} sudo systemctl start tomcat
                    """
                }
            }
        }
        stage("Deploy War File To QA Server"){
            
            
            when {
                expression { env.BRANCH_NAME ==  "QA" }
            }

            steps {
                sshagent(['TomcatServer_SSH_Credetails']) {
                  sh """
                    echo "Deploying to QA Server"
                       ssh -o StrictHostKeyChecking=no ${tomcatSSHusername}@${tomcatIP} sudo systemctl stop tomcat || true
                       sleep 20
                       ssh -o StrictHostKeyChecking=no ${tomcatSSHusername}@${tomcatIP} rm -f /opt/tomcat/webapps/student-reg-webapp.war
                       scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ${tomcatSSHusername}@${tomcatIP}:/opt/tomcat/webapps/student-reg-webapp.war
                       ssh -o StrictHostKeyChecking=no ${tomcatSSHusername}@${tomcatIP} sudo systemctl start tomcat
                    """
                }
            }
        }
  
       stage("Deploy War File To Prod Server"){
             
            when {
                expression { env.BRANCH_NAME ==  "main" }
            }
            steps {
              sshagent(['TomcatServer_SSH_Credetails']) {
                sh """
                  echo "Deploying to Prod Server"
                    """
              }
            }
        }
    }
    post {
        success {
            slackSend channel: '#all-devops-pratice',color: "good", message: "Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER} - ${env.BUILDSTATUS} - Success. Check console output at ${env.BUILD_URL}"
        }
        failure {
            slackSend channel: '#all-devops-pratice',color: "denger", message: "Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER} - ${env.BUILDSTATUS} - Failure. Check console output at ${env.BUILD_URL}"
        }
        always {
            cleanWs()
        }
    }
}
