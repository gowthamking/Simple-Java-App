pipeline {
    agent {
        node {
            lable 'maven'
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
    }
}