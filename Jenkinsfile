pipeline {
  agent any
  stages {
    stage('Execute UnitTest') {
      steps {
        sh " cd tests/UnitTests/ && dotnet test"
      }
    }
    stage('Execute Integration Test') {
      steps {
        sh " cd  tests/IntegrationTests/ && dotnet test"
      }
    }
    stage('Docker Build') {
      steps {
        sh "sudo docker-compose build"
        sh "sudo docker ps --filter 'label=name=Demo_App' -q | xargs --no-run-if-empty sudo docker container stop"
        sh "sudo docker ps --filter 'label=name=Demo_App' -q | xargs -r sudo docker container rm"
        sh "sudo docker-compose up -d"
      }
    }
    stage('Clone the repo of Testng-Cucumber') {
      steps {
        sh "rm -rf testng-cucumber && mkdir testng-cucumber && cd testng-cucumber/ && git clone https://github.com/wissen-qa/testng-cucumber.git && cd testng-cucumber && ls -ltrh"
        sh "pwd && cd testng-cucumber/testng-cucumber && export PATH=$PATH:/opt/apache-maven-3.5.3/bin && mvn clean test"
      }
    }
    stage('Docker Remove Image') {
      steps {
        sh "docker image prune -a"
      }
    }
}
post {
    success {
      slackSend(message: "Pipeline is successfully completed.")
    }
    failure {
      slackSend(message: "Pipeline failed. Please check the logs.")
    }
}
}
