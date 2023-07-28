pipeline {
    agent any

    tools {
        maven 'Maven3_5_2'
    }
     environment {
        AWS_REGION = 'us-west-2'
        ECR_REPO_URL = 'https://079084503647.dkr.ecr.us-west-2.amazonaws.com/benlmaoujoud'
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

	    stage('Test AWS Credentials') {
    steps {
        script {
            def awsCredentials = credentials('aws-credentials') // Replace 'aws-credentials' with the ID of your Jenkins credentials containing AWS access key and secret key
            
            // Execute the AWS CLI command to retrieve the ECR login token
            def awsLoginCommand = "aws ecr get-login-password --region us-west-2"
            def awsToken = sh(script: awsLoginCommand, returnStdout: true).trim()
            

            if (awsToken) {
                echo "AWS credentials test passed. Successfully authenticated with ECR."
            } else {
                error "AWS credentials test failed. Unable to authenticate with ECR."
            }
        }
    }
}


	stage('Push') {
    steps {
        script {
            def services = sh(
                script: 'docker-compose config --services',
                returnStdout: true
            ).trim().split('\n')

            def ecrRepoUrl = env.ECR_REPO_URL // Get the value of the ECR_REPO_URL environment variable

            docker.withRegistry(ecrRepoUrl, 'ecr:us-west-2:aws-credentials') {
                services.each { service ->
                    def sourceImage = "${service}:latest"
                    def targetImage = "${ecrRepoUrl}/${service}:latest"

                    sh "docker tag ${sourceImage} ${targetImage}"
                    sh "docker push ${targetImage}"
                }
            }
        }
    }
}



    }
}
