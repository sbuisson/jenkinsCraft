void setBuildStatus = (String url, String context, String message, String state, String backref){ ->
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
  sh "git config --get remote.origin.url > originurl"
  return readFile("originurl").trim()
}

pipeline {
    agent anygogits/go-gogs-client
    stages {
        stage('Build') {
            steps {
               echo 'This is a minimal pipeline.'
               echo 'This is also minimal pipeline.'
               def repoUrl = getRepoURL()
               echo repoUrl
               sh 'mvn clean install'
               setBuildStatus(repoUrl, "ci/approve", "Aprove after testing", "PENDING", "")
            }
        }
    }
}