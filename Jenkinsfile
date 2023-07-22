pipeline {
  agent any
  tools {
        maven 'Maven3_5_2'
    }
   stages{
    stage('Compile and Run SonarAnalysis') {
            steps {
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=ensias-internship -Dsonar.organization=ensias-internship -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=87868ea1485745b552e22260f9635d4bbab65d6c -DskipTests'
			}
        }
	
	    stage('Snyk Security Scan') {
            steps {
                script {
                    // Run Snyk security scan for each microservice
                    def microservices = ['product-service', 'order-service', 'inventory-service', 'notification-service']

                    for (def microservice : microservices) {
                        dir("${microservice}") {
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}                        }
                    }
                }
            }
        }
  }
}
