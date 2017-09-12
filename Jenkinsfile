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
                /* if (env.BRANCH_NAME.startsWith('PR')) {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'sbuisson-git', usernameVariable: 'GH_LOGIN', passwordVariable: 'GH_PASSWORD']]) {
                        def resp = httpRequest url: "https://api.github.com/repos/sbuisson/jenkinsCraft/pulls/${env.BRANCH_NAME.substring(3)}", customHeaders: [[name: 'Authorization', value: "token ${env.GH_PASSWORD}"]]
                        def ttl = getTitle(resp)
                        def itm = getItem(env.BRANCH_NAME)
                        itm.setDisplayName("PR-${env.BRANCH_NAME.substring(3)} '${ttl}'")
                    }
                }*/
                checkout scm
                withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                    if ("master" == env.BRANCH_NAME) {
                        sh "mvn clean install"
                    } else {
                        sh "mvn clean install"
                    }
                }
                archiveArtifacts artifacts: 'target/*.hpi'
                    // step([$class: 'Publisher', reportFilenamePattern: '**/testng-results.xml'])
                echo "sonar"
                if ("master" == env.BRANCH_NAME) {
                    withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                        echo "sonar master"
                        sh "mvn -Dsonar.host.url=http://sonarqube:9000 sonar:sonar"
                    }
                } else {
                    withEnv(["PATH+MAVEN=${tool name: 'Maven 3', type: 'hudson.tasks.Maven$MavenInstallation'}/bin"]) {
                        echo "sonar branch"
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'sbuisson-git', usernameVariable: 'GH_LOGIN', passwordVariable: 'GH_PASSWORD']]) {
                            echo "sonar branch"
                            echo "sonar branch"
                            sh "mvn -Dsonar.host.url=http://sonarqube:9000 -Dsonar.analysis.mode=issues -Dsonar.github.pullRequest=${env.BRANCH_NAME.substring(3)} -Dsonar.github.repository=sbuisson/jenkinsCraft -Dsonar.github.login=${env.GH_LOGIN} -Dsonar.github.password=${env.GH_PASSWORD} sonar:sonar mvn sonar:sonar \
  -Dsonar.host.url=http://sonarqube:9000 \
  -Dsonar.login=admin \
  -Dsonar.password=admin                        "
                        }
                    }
                }
}




            }
        }
    }
}

