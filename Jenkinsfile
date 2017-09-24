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
         stage('send Message') {
            steps {

script{
def SHA1 ='3333'
              //  def SHA1 = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
                //echo '$SHA1'

              //  httpRequest authentication: 'sbuisson-git', httpMode: 'GET',  url: 'https://api.github.com/sbuisson/jenkinsCraft/pulls/2/comments'


              /*  httpRequest authentication: 'sbuisson-git', httpMode: 'POST', requestBody: '{\
                    "body": "Nice change",\
                    "commit_id": "$SHA1",\
                    "path": "./",\
                    "position": 0\
                }',  url: 'https://api.github.com/sbuisson/JenkinsCraft/Hello-World/pulls/1347/comments'
                */
                }

            }
        }
        stage('repoUrl') {
            steps {


                 echo repoUrl
                  }
        }

        stage('docker') {
            steps {
                checkout scm
                withDockerContainer(image:'maven:3.3.3-jdk-8', args:"-v  ${pwd()}/workspaceBis:/data") {

                            echo "docker, baby!"
                            sh "pwd"
                            sh "mvn -v"
                            sh 'mvn clean install'

                }
                    sh "pwd"
                    sh "ls -lrt"

            }

        }
        stage('status') {
            steps {
                echo "status for ${env.BRANCH_NAME.substring(3)}"
                setBuildStatus("https://github.com/sbuisson/jenkinsCraft/", "ci/approve", "Aprove after testidsng", "PENDING", "")


            }
        }


        stage('analyse') {
            steps {
                checkout scm

                script {
                    sh "mvn -v"
                    docker
                        .image('maven:3.3.3-jdk-8')
                        .inside("-v  ${pwd()}/workspaceTer:/data") {


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
                                 if ("master" == env.BRANCH_NAME) {
                                    sh "mvn clean install"
                                } else {
                                    sh "mvn clean install"

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


                                echo "sonar branch ${env.GH_LOGIN}"

                                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'sbuisson-git', usernameVariable: 'GH_LOGIN', passwordVariable: 'GH_PASSWORD']]) {
                                    withCredentials([[$class: 'StringBinding', credentialsId: ' git-token', variable: 'OATH']]) {
 sh "mvn -v"
                                        echo "sonar branch"
                                        echo "sonar branch"
                                        sh "mvn pitest:mutationCoverage \
                                            -Dsonar.host.url=http://sonarqube:9000\
                                            -Dsonar.analysis.mode=preview\
                                            -Dsonar.github.pullRequest=${env.BRANCH_NAME.substring(3)}\
                                            -Dsonar.github.repository=sbuisson/jenkinsCraft \
                                            -Dsonar.github.login=${env.GH_LOGIN} -Dsonar.github.password=${env.GH_PASSWORD} \
                                            -Dsonar.github.oauth=${env.OATH} -Dsonar.pitest.mode=reuseReport \
                                            sonar:sonar \
                                            -Dsonar.host.url=http://sonarqube:9000 \
                                            -Dsonar.login=admin \
                                            -Dsonar.password=admin "
                                    }
                    
                                }
                            }
                        }

                }
            }
        }
    }
}

