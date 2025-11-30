pipeline {
  agent any
  tools { 
        maven 'Maven_3_9_11'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=asgbuggywebapp710_sai -Dsonar.organization=asgbuggywebapp710 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=3656e6e833eefe7f5b28f4dd9b28fa016c0e6c83'
			}
    }
	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'Snyk_token', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }
	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://140023360548.dkr.ecr.us-east-1.amazonaws.com/asg', 'ecr:us-east-1:aws-credentials') {
                    app.push("latest")                                                                        
                    }
                }
            }
    	}
	   stage('Kubernetes Deployment of ASG Bugg Web Application') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {

      // Generate fresh kubeconfig that contains a valid IAM token
            sh '''
              aws eks update-kubeconfig \
                --name kubernetes-cluster \
                --region us-east-1
            '''

            sh 'kubectl get nodes'
            sh 'kubectl delete all --all -n devsecops || true'
            sh 'kubectl apply -f deployment.yaml --namespace=devsecops'
    }
  }
}

	   
	stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 



   }
}
