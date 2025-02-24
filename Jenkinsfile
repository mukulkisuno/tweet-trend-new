def registry = 'https://mukulkisuno.jfrog.io/'
def imageName = 'mukulkisuno.jfrog.io/mukulkisuno-docker-local/ttrend'
def version   = '2.1.3'
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
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifact-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "maven-libs-release-local/{1}",
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
                docker.withRegistry(registry, 'artifact-cred'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }
stage(" Deploy package ttrend via Helm chart") {
       steps {
         script {
            echo '<--------------- Helm Deploy Started --------------->'
            sh 'helm install ttrend ttrend-0.1.0.tgz'
            echo '<--------------- Helm deploy Ends --------------->'
         }
       }
     }
}
}

