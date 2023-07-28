pipeline {
    agent any

    tools {
        maven 'Maven3_5_2'
    }

    stages {
        stage('Compile and Run SonarAnalysis') {
            steps {
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=ensias-internship -Dsonar.organization=ensias-internship -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=87868ea1485745b552e22260f9635d4bbab65d6c -DskipTests'
            }
        }

        stage('Snyk Scan') {
            steps {
       
                       withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					            sh 'cd product-service && mvn snyk:test -fn'
				        }
                    
                
            }
        }
	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 sh 'docker-compose build'
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://145988340565.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                    sh 'docker-compose push'
                    }
                }
            }
    	}

    }
}
