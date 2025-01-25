pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
}
    stages {
        stage ("build"){
            steps {
                echo "--------------build started-----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "--------------build completed-----------"
            }
        }
        stage ("test"){
            steps{
                echo "--------------unit test started-----------"
                sh 'mvn surefire-report:report'
                echo "--------------unit test completed-----------"
            }
        }
        stage('SonarQube analysis') {
        environment {
            scannerHome = tool 'mukulkisuno-sonar-scanner';
        }
        steps {
        withSonarQubeEnv('mukulkisuno-sonarqube-server') {
            sh """
                ${scannerHome}/bin/sonar-scanner -X \
                -Dsonar.projectKey=mukulkisuno01-key_twittertrend \
                -Dsonar.organization=mukulkisuno01-key \
                -Dsonar.sources=./src \
                -Dsonar.host.url=https://sonarcloud.io/ \
                -Dsonar.login=3602b425dfa750fcbe1525981b49f27757d52cfd \
                -Dsonar.java.binaries=target/classes
            """
        }
    }
  }
}
}
