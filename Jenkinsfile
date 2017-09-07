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



   def getItem(branchName) {
     Jenkins.instance.getItemByFullName("sonar-openedge/${branchName}")
   }


   def getTitle(json) {
       def slurper = new groovy.json.JsonSlurper()
       def jsonObject = slurper.parseText(json.content)
       jsonObject.title
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

                sh 'mvn clean install'
                 }


        }
        stage('status') {
                     steps {
                         setBuildStatus(repoUrl, "ci/approve", "Aprove after testing", "PENDING", "")


                    }
                }


        stage('analyse') {
            steps {
                script{


                // Set job description with PR title
                if (env.BRANCH_NAME.startsWith('PR')) {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'git-xebia', usernameVariable: 'GH_LOGIN', passwordVariable: 'GH_PASSWORD']]) {
                        def resp = httpRequest url: "https://api.github.com/repos/sbuisson/jenkinsCraft/pulls/${env.BRANCH_NAME.substring(3)}", customHeaders: [[name: 'Authorization', value: "token ${env.GH_PASSWORD}"]]
                        def ttl = getTitle(resp)
                        def itm = getItem(env.BRANCH_NAME)
                        itm.setDisplayName("PR-${env.BRANCH_NAME.substring(3)} '${ttl}'")
                    }
                }
                checkout scm
                withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                    if ("master" == env.BRANCH_NAME) {
                        sh "mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install"
                    } else {
                        sh "mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent package"
                    }
                }
                archiveArtifacts artifacts: 'target/*.hpi'
                    // step([$class: 'Publisher', reportFilenamePattern: '**/testng-results.xml'])

                if ("master" == env.BRANCH_NAME) {
                    withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                        sh "mvn -Dsonar.host.url=http://sonar.riverside-software.fr sonar:sonar"
                    }
                } else {
                    withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'GitHub-GQuerret', usernameVariable: 'GH_LOGIN', passwordVariable: 'GH_PASSWORD']]) {
                            sh "mvn -Dsonar.host.url=http://sonar.riverside-software.fr -Dsonar.analysis.mode=issues -Dsonar.github.pullRequest=${env.BRANCH_NAME.substring(3)} -Dsonar.github.repository=jakejustus/openedge-jenkins-public -Dsonar.github.oauth=${env.GH_PASSWORD} sonar:sonar"
                        }
                    }
                }
}




            }
        }
    }
}

