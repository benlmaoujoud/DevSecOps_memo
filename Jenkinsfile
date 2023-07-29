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
                  input message: 'Confirm Docker System Prune', parameters: [booleanParam(defaultValue: true, description: 'Confirm Docker System Prune', name: 'confirm')]
                  sh 'docker system prune -a --volumes'

                    sh 'docker-compose build'
                    sh 'docker images'
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
                def SERVICES = [
                        'notification-service',
                        'product-service',
                        'order-service',
                        'inventory-service',
                        'discovery-server',
                        'api-gateway'
                    ]

                if (!ecrRepoUrl.startsWith('https://')) {
                    error "Invalid ECR_REPO_URL: The URL must start with 'https://'"
                }

                docker.withRegistry("https://${ecrRepoUrl}", 'ecr:us-west-2:aws-credentials') {
                    SERVICES.each { service ->
                        def targetImage = "${ecrRepoUrl}/benlmaoujoud:latest"
                        sh "docker tag ${service} ${targetImage}"
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

