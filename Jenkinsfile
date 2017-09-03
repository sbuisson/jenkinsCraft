void setBuildStatus(String url, String context, String message, String state, String backref){
  step([
    $class: "GitHubCommitStatusSetter",
    reposSource: [$class: "ManuallyEnteredRepositorySource", url: url ],
    contextSource: [$class: "ManuallyEnteredCommitContextSource", context: context ],
    errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
    statusBackrefSource: [ $class: "ManuallyEnteredBackrefSource", backref: backref ],
    statusResultSource: [ $class: "ConditionalStatusResultSource", results: [
        [$class: "AnyBuildResult", message: message, state: state]] ]
  ]);
}

def getRepoURL = {
  sh "git config --get remote.origin.url"
  sh "git config --get remote.origin.url > originurl"
  def originurl = readFile("originurl").trim()
  return originurl
}
def repoUrl = "orignalRepoURL"
pipeline {
    agent any
    stages {

        stage('Build') {
            steps {

               echo 'This is a minimal pipeline.'
            }
        }
         stage('Build2') {
            steps {


               echo 'This is also minimal pipeline.'


            }
        }
        stage('repoUrl') {
            steps {


                 echo repoUrl
                  }
        }
        stage('mvn') {
            steps {
                def mvnContainer = docker.image('jimschubert/8-jdk-alpine-mvn')

                script {
                    mvnContainer.inside('-v /m2repo:/m2repo') {
                        // Build with maven settings.xml file that specs the local Maven repo.
                        sh 'mvn clean install'
                    }
                 }
            }

        }
        stage('status') {
                     steps {
                         setBuildStatus(repoUrl, "ci/approve", "Aprove after testing", "PENDING", "")


                    }
                }
    }
}