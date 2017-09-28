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

String getRepoURL() {
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
   void sendCommentToPullRequest(String messageContent){

         def SHA1 ="SHA1"
         script {
            SHA1 = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
         }
         def message = """{"body": "$messageContent", "commit_id": "$SHA1", "path": "/", "position": 0}"""
         httpRequest authentication: 'sbuisson-git', httpMode: 'POST', requestBody: "${message}",  url: "https://api.github.com/repos/sbuisson/jenkinsCraft/issues/2/comments"
   }



node {


/*
        stage('repoUrl') {
            steps {
script {
                 def repoUrl = ''
                  }
                 repoUrl = getRepoURL()
                 echo repoUrl

              }

        }
        stage('send Message') {

           echo "hy"

            sendCommentToPullRequest("message")

              echo "message Send"

        }
*/

        stage('docker') {

                script{
                    checkout scm
                    withDockerContainer(image:'maven:3.3.3-jdk-8', args:"-v  ${pwd()}/workspaceBis:/data") {

                                echo "docker, baby!"
                                sh "mvn -v"
                                sh 'mvn clean install -B'


                    }
                }
              sendCommentToPullRequest( "build docker done")

        }
        stage('status') {

                echo "status for ${env.BRANCH_NAME.substring(3)}"
                setBuildStatus("https://github.com/sbuisson/jenkinsCraft/", "ci/approve", "Aprove after testidsng", "PENDING", "")



        }


        stage('analyse') {

                checkout scm

                script {
                    sh "mvn -v"

                    docker
                        .image('maven:3.3.3-jdk-8')
                        .inside("-v workspaceTer:/data") {

                            checkout scm
                                 if ("master" == env.BRANCH_NAME) {
                                    sh "mvn clean install -B"
                                } else {
                                    sh "mvn clean install -B"

                                }
                                sh "pwd"
                            }
                    sh "ls"
                    def databaseSonarParam = " -Dsonar.jdbc.username=ci_user -Dsonar.jdbc.password=ci -Dsonar.jdbc.url=jdbc:postgresql://postgres:5432/ci "
                    def sonarParam = " -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=admin -Dsonar.password=admin "

                    echo "sonar"
                    if ("master" == env.BRANCH_NAME) {
                        withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                            echo "sonar master"
                                  sh "mvn sonar:sonar $sonarParam $databaseSonarParam"

                        }
                    } else {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'sbuisson-git', usernameVariable: 'GH_LOGIN', passwordVariable: 'GH_PASSWORD']]) {
                            withCredentials([[$class: 'StringBinding', credentialsId: ' git-token', variable: 'OATH']]) {
                                sh "mvn -v"
                                echo "sonar branch"
                                sh "mvn sonar:sonar $sonarParam $databaseSonarParam"
                                echo "sonar branch ${env.GH_LOGIN}"
                                def githubSonarParam="-Dsonar.github.pullRequest=${env.BRANCH_NAME.substring(3)}\
                                    -Dsonar.github.repository=sbuisson/jenkinsCraft \
                                    -Dsonar.github.login=${env.GH_LOGIN} -Dsonar.github.password=${env.GH_PASSWORD} \
                                    -Dsonar.github.oauth=${env.OATH} "

                                def mvnQuery= "mvn pitest:mutationCoverage  sonar:sonar \
                                   $sonarParam $databaseSonarParam $githubSonarParam \
                                    -Dsonar.analysis.mode=preview -Dsonar.pitest.mode=reuseReport"
                                sh mvnQuery

                                archive "target/sonar/**/*"

                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site', reportFiles: 'index.html', reportName: 'HTML site', reportTitles: 'a'])
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/pit-reports', reportFiles: 'index.html', reportName: 'HTML site', reportTitles: 'b'])
                                sendCommentToPullRequest( "fin <a href='http://localhost:8080/job/sbuisson/job/jenkinsCraft/view/change-requests/job/PR-2/131/artifact/target/site/index.html'>report</a>")


                                      }
                    

                            }
                        }




                }

            }

            stage("archive"){



            }



}

