pipeline{
  agent {
	  label 'jenkins-slave'
  }
  parameters {
	  choice(name: 'BUILD', choices: ['One', 'Two'], description: 'Choose Build')
  }
  stages{
	  stage('Checkout code'){
		  steps{
			  git 'https://github.com/KevLeng/Workshop-AC-WithKeptn.git'
			  sh 'ls ' + env.WORKSPACE
		  }
	  }
	  stage('Build Code') {
		  steps{
			  script{
				  echo 'Code Building...'
			  }
		  }
	  }
	  stage('Build Image'){
		  steps{
			  script{
				  echo 'Image Building...'
			  }
		  }
	  }
	  stage('Push Image to Repo'){
		  steps{
			  script{
				  echo 'Pushing Image...'
			  }
		  }
	  }
	  stage('Deploy to Staging'){
		  steps{
			  container('kubectl'){
				  echo 'Deployment canary build...'
				  //sh 'ls ' + env.WORKSPACE
				  script{
					  if (params.BUILD == 'One') {
						  sh 'ls ' + env.WORKSPACE
						  sh 'kubectl apply -f ' + env.WORKSPACE + '/manifests/sockshop-app/dev/carts2.yml'
					  } else {
						  sh 'kubectl apply -f ' + env.WORKSPACE + '/manifests/sockshop-app/canary/carts2-canary.yml'
					  }
					  echo 'Waiting for carts service to start...'
					  sleep 10
					  
					  def cartsIp = sh (returnStdout:true, script:"kubectl get ingress dev-ingress -n dev  | grep -o carts.dev.*io,").trim()
					  cartsIp = cartsIp.minus(",")
					  echo "Carts IP Address is: " + cartsIp
					  
					  def serviceResponse = ""
					  timeout(time: 10, unit: 'MINUTES') {
						  script {
							  waitUntil {
								  // get carts url 
								  def response = httpRequest httpMode: 'GET', 
									  responseHandle: 'STRING', 
									  url: 'http://' + cartsIp + '/', 
									  validResponseCodes: "100:505"
				  
								  //The API returns a response code 500 error if the evalution done event does not exisit
								  if (response.status != 200 ) {
									  echo 'Waiting for carts service to start, ip address: ' + cartsIp
									  sleep 20
									  return false
								  } else {
									  serviceResponse = response.content
									  echo "Carts service started."
									  return true
								  } 
							  }
						  }
					  }
				  }
			  }
		  }
	  }
	  stage ('Run Load Test'){
		  steps{
			  container('kubectl'){
				  echo 'Get Carts URL'
				  script{
					  def cartsIp = sh (returnStdout:true, script:"kubectl get ingress dev-ingress -n dev  | grep -o carts.dev.*io,").trim()
					  cartsIp = cartsIp.minus(",")
					  echo "Carts IP Address is: " + cartsIp
					  dir('loadtest'){
						  sh 'chmod +x cartstest.sh'
						  sh './cartstest.sh ' + cartsIp

					  }
				  }
			  }
			  container('jmeter'){
				  script{
					  if (params.BUILD == 'One'){
						  dir('loadtest'){
							  sh 'jmeter -n -t carts_load1.jmx -l testresults.jtl'
						  }   
					  } else {
						  dir('loadtest'){
							  sh 'jmeter -n -t carts_load2.jmx -l testresults.jtl'
						  } 
					  }

				  }
			  }
		  }
	  }
	  stage ('Deploy to Production'){
		  steps{
			  container('kubectl'){
				  echo 'Deploying to Production'
				  script{
					  if (params.BUILD == 'One') {
						  echo 'Waiting for carts service to start...'
						  sleep 20
					  } else {
						  echo 'Waiting for carts service to start...'
						  sleep 20
					  }
				  }
			  }
		  }
	  }
  }
}