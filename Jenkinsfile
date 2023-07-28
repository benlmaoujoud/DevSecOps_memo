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
	stage('Build and Push Docker Images') {
            steps {
                sh 'mvn clean package -DskipTests'
                sh 'docker-compose build'
                script {
                    def ecrRegistry = '079084503647.dkr.ecr.us-west-2.amazonaws.com'
                    def awsCredentials = 'ecr:us-west-2:aws-credentials'
                    sh "docker-compose push --push-option=aws_region=us-west-2 --push-option=aws_credentials=${awsCredentials} --push-option=aws_ecr_registry=${ecrRegistry}"
                }
            }
        }

    }
}
