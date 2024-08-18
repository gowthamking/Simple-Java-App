def registry = 'https://gowzip.jfrog.io/'
def imageName = 'gowzip.jfrog.io/test-docker-local/simplejava'
def version   = '2.1.3'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }

environment {
    PATH = "/opt/apache-maven-3.9.8/bin:$PATH"
}
    stages{
        stage("Build"){
            steps{
                echo "------------------ Build Started --------------------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "------------------ Build Completed --------------------------"
            }
        }

        stage("Unit Test"){
            steps{
                echo "------------------ Unit Test Started --------------------------"
                sh 'mvn surefire-report:report'
                echo "------------------ Unit Test Started --------------------------"
            }
        }
    
       stage('SonarQube analysis') {

       environment{
          scannerHome = tool 'test-sonar-scanner'
          }

           steps{

            withSonarQubeEnv('test-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
               sh "${scannerHome}/bin/sonar-scanner"
               }

               }
            }


        stage("Quality Gate") {
            steps {
                script {
                  echo '<--------------- Sonar Gate Analysis Started --------------->'
                    timeout(time: 1, unit: 'HOURS'){
                       def qg = waitForQualityGate()
                        if(qg.status !='OK') {
                            error "Pipeline failed due to quality gate failures: ${qg.status}"
                        }
                    }  
                  echo '<--------------- Sonar Gate Analysis Ends  --------------->'
                }
            }
        }


        stage("Jar Publish") {
              steps {
                  script {
                          echo '<--------------- Jar Publish Started --------------->'
                           def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-artifact-cred"
                           def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                           def uploadSpec = """{
                                "files": [
                                  {
                                    "pattern": "jarstaging/(*)",
                                    "target": "test-libs-release-local/{1}",
                                    "flat": "false",
                                    "props" : "${properties}",
                                    "exclusions": [ "*.sha1", "*.md5"]
                                  }
                               ]
                           }"""
                           def buildInfo = server.upload(uploadSpec)
                           buildInfo.env.collect()
                           server.publishBuildInfo(buildInfo)
                           echo '<--------------- Jar Publish Ended --------------->'  
                  
                  }
              }   
          }


        stage(" Docker Build ") {
              steps {
                script {
                   echo '<--------------- Docker Build Started --------------->'
                   app = docker.build(imageName+":"+version)
                   echo '<--------------- Docker Build Ends --------------->'
                }
              }
            }
        
        stage (" Docker Publish "){
                steps {
                    script {
                       echo '<--------------- Docker Publish Started --------------->'  
                        docker.withRegistry(registry, 'jfrog-artifact-cred'){
                            app.push()
                        }    
                       echo '<--------------- Docker Publish Ended --------------->'  
                    }
                }
            }

        stage(" Deploy ") {
          steps {
            script {
               echo '<--------------- Deploy Started --------------->'
            //    sh 'helm uninstall simplejava || true'
            //    sh 'helm install simplejava simplejava-0.1.0.tgz'
               sh 'helm upgrade --install simplejava simplejava-0.1.0.tgz --recreate-pods'
               echo '<--------------- Deploy Ends --------------->'
            }
          }
        } 

    post {
 always {
    script {
    def jobName = env.JOB_NAME
    def buildNumber = env.BUILD_NUMBER
    def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
    def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
 
    def body = """<html>
                    <body>
                        <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                        <h2>${jobName} - Build ${buildNumber}</h2>
                        <div style="background-color: ${bannerColor}; padding: 10px;">
                        <h3 style="color: white;">Pipeline Status: 
                        ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
            </body>
        </html>"""


    emailext (
        subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
        body: body,
        to: 'gowtham.zippy@gmail.com',
        from: 'jenkins@example.com',
        replyTo: 'jenkins@example.com',
        mimeType: 'text/html'
 )
 }
 }
}


        }


}
