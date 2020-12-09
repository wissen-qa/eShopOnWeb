pipeline {
  agent any
  stages {
    stage('Execute UnitTest') {
      steps {
        sh " cd tests/UnitTests/ && dotnet test"
      }
    }
    stage('CodeCoverage and Static Code Report') {
      steps {
        sh "export PATH=$PATH:/home/BuildMachine/.dotnet/tools/ && dotnet sonarscanner begin /k:e09a98e2b9b267b2086c33c4cf1d40750e51f072 /d:sonar.host.url=http://20.55.120.136:9000 && dotnet build eShopOnWeb.sln && dotnet sonarscanner end" 
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
        sh "sudo docker image prune -a --force"
      }
    }
  }
	post {
		always {
			script {
				CONSOLE_LOG = "${env.BUILD_URL}/console"
				BUILD_STATUS = currentBuild.currentResult
				if (currentBuild.currentResult == 'SUCCESS') {
					CI_ERROR = "NA"
					}
				}
				sh """
					TODAY=`date +"%b %d"`
					sed -i "s/%%JOBNAME%%/${env.JOB_NAME}/g" report.html
					sed -i "s/%%BUILDNO%%/${env.BUILD_NUMBER}/g" report.html
					sed -i "s/%%DATE%%/\${TODAY}/g" report.html
					sed -i "s/%%BUILD_STATUS%%/${BUILD_STATUS}/g" report.html
					sed -i "s/%%ERROR%%/${CI_ERROR}/g" report.html
					sed -i "s|%%CONSOLE_LOG%%|${CONSOLE_LOG}|g" report.html
				"""
				publishHTML(target:[
					allowMissing: true,
					alwaysLinkToLastBuild: true, 
					keepAll: true, 
					reportDir: "${WORKSPACE}", 
					reportFiles: 'report.html', 
					reportName: 'CI-Build-HTML-Report', 
					reportTitles: 'CI-Build-HTML-Report'
					])
				sendSlackNotifcation()
		}
	}
}

def sendSlackNotifcation() 
{ 
	if ( currentBuild.currentResult == "SUCCESS" ) {
		buildSummary = "Job:  ${env.JOB_NAME}\n Status: *SUCCESS*\n Build Report: ${env.BUILD_URL}CI-Build-HTML-Report"

		slackSend color : "good", message: "${buildSummary}", channel: '#test-slack'
		}
	else {
		buildSummary = "Job:  ${env.JOB_NAME}\n Status: *FAILURE*\n Error description: *${CI_ERROR}* \nBuild Report :${env.BUILD_URL}CI-Build-HTML-Report"
		slackSend color : "danger", message: "${buildSummary}", channel: '#test-slack'
		}
}

