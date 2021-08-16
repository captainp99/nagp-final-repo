pipeline{
  agent any
  environment {
    gitHubUrl= 'https://github.com/captainp99/nagp-final-repo'
	scannerHome= tool name: 'sonar_scanner_dotnet'
	username= 'parasjain01'
	registry= 'paras22/nagp-devops-assign-1'
  }
  
  stages{
    stage('Checkout git code'){
	   steps{
	     echo 'Checking out code start'
	     git url: env.gitHubUrl
	   }
	}
	
	stage('Start Sonarqube analysis') {
	  steps{
	     echo 'Start Sonarqube analysis'
		 withSonarQubeEnv("Test_Sonar"){
		  bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-${username} -d:sonar.cs.opencover.reportsPath=test-project/coverage.opencover.xml -d:sonar.cs.xunit.reportsPaths='test-project/TestResults/TestFileReport.xml'"
		 }
	  }
	}
	
	stage('Buid Code'){
	 steps{
	   echo 'Clean Previous Build'
	   bat 'dotnet clean'
	   
	   echo 'Build Solution'
	   bat 'dotnet build -c Release'
	 }
	}
	
	stage('Execute Test Cases') {
	   steps{
	      echo "Test Case Execution"
		  bat "dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestFileReport.xml"
	   }
	   
	}
	
	stage('Stop Sonarqube analysis') {
	  steps{
	     echo 'Stop Sonarqube analysis'
		 withSonarQubeEnv("Test_Sonar"){
		  bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
		 }
	  }
	}
	
	stage('Build Docker Image') {
	  steps{
	     echo "Build Docker"
		 bat "docker build -t i-${username}-feature ."
		 bat "docker tag i-${username}-feature ${registry}-feature:$BUILD_NUMBER"
		 bat "docker tag i-${username}-feature ${registry}-feature:latest"
	  }
	}
	
	stage('Push Image to dockerhub') {
	  steps{
	     echo "Push Image to dockerhub"
	     withDockerRegistry(credentialsId: 'DockerHub', url: ''){
		   bat "docker push ${registry}-feature:$BUILD_NUMBER"
		   bat "docker push ${registry}-feature:latest"
		 }
	  }
	}
	
	stage('Pre-container check'){
	  steps{
	    script{
			  echo "Precontainer check"
			  checkIfContainerALreadyExist = bat(script: "docker ps -qa -f name=c-${username}-feature", returnStdout: true).trim().readLines().drop(1).join("")
			  if(checkIfContainerALreadyExist){
			    echo "Container already exist"
				checkIfRunning = bat(script: "docker ps -q -f name=c-${username}-feature", returnStdout: true).trim().readLines().drop(1).join("")
				if(checkIfRunning) {
				  echo "Stopping Container "
				  bat "docker stop c-${username}-feature"
				}
				echo "Removing Container"
				bat "docker rm c-${username}-feature"
			  }
			}
	  }
	}
	
	stage('Deployment') {
	  steps{
	     parallel(
		    "Docker Deployment": {
			   bat "docker run --name c-${username}-feature -d -p 7400:80 i-${username}-feature"
			},
			"Kuberneted Deployment": {
			   bat "kubectl apply -f deployment.yaml"
			}
		 )
	  }
	}
  }
}
