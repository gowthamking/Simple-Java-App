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
                sh 'mvn clean deploy'
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
        }


}
