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
        }


}
