Testing_Repo/testng-cucumberpipeline {
  agent any
  stages {
    stage('Running UITest') {
      steps {
        sh "git clone https://github.com/wissen-qa/testng-cucumber.git && cd testng-cucumber && pwd && mvn clean test"
      }
    }
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
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push kmlaydin/podinfo:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Docker Remove Image') {
      steps {
        sh "docker rmi kmlaydin/podinfo:${env.BUILD_NUMBER}"
      }
    }
    stage('Apply Kubernetes Files') {
      steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'cat deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
          sh 'kubectl apply -f service.yaml'
        }
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
