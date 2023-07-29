pipeline {
    agent any

    tools {
        maven 'Maven3_5_2'
    }
    
    environment {
        AWS_REGION = 'us-west-2'
        ECR_REPO_URL = '079084503647.dkr.ecr.us-west-2.amazonaws.com'
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
                withDockerRegistry([credentialsId: 'dockerlogin', url: '']) {
                    sh 'docker-compose build'
                }
            }
        }

        stage('Test AWS Credentials') {
            steps {
                script {
                    def awsCredentials = credentials('aws-credentials')

                    def awsLoginCommand = "aws ecr get-login-password --region ${env.AWS_REGION}"
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
                    try {
                        def ecrRepoUrl = env.ECR_REPO_URL.trim()
                        
                        def services = sh(
                            script: 'docker-compose config --services',
                            returnStdout: true
                        ).trim().split('\n')

                        docker.withRegistry(ecrRepoUrl, 'ecr:us-west-2:aws-credentials') {
                            services.each { service ->
                                def sourceImage = "${service}:latest"
                                def targetImage = "${ecrRepoUrl}/benlmaoujoud/${service}:latest" 

                                sh "docker tag ${sourceImage} ${targetImage}"
                                sh "docker push ${targetImage}"
                            }
                        }
                    } catch (Exception e) {
                        error "Failed to push Docker images to ECR: ${e.message}"
                    }
                }
            }
        }
   

    }
}
