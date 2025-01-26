def registry = 'https://mukulkisuno.jfrog.io/'
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
    stage("Quality Gate") 
    {
            steps {
                script{
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
  }
}
     
stage("Jar Publish") {
    steps {
        script {
            echo '<--------------- Jar Publish Started --------------->'
            def server = Artifactory.newServer(url: "https://mukulkisuno.jfrog.io/artifactory", credentialsId: "artifact-cred")
            def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
            
def uploadSpec = """
{
    "files": [
        {
            "pattern": "jarstaging/*",
            "target": "maven-libs-release-local/",
            "flat": "false",
            "props": "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}",
            "exclusions": ["*.sha1", "*.md5"],
            "overwrite": true
        }
    ]
}
"""
            
            def buildInfo = server.upload(uploadSpec)
            buildInfo.env.collect()
            server.publishBuildInfo(buildInfo)
            echo '<--------------- Jar Publish Ended --------------->'
        }
    }
}
}
}

