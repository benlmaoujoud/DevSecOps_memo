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
                    echo "@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ ALL images @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@#...."
                    sh 'docker-compose config --services'
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

       stage('Push to ECR') {
    steps {
        script {
            def ecrRepoUrl = env.ECR_REPO_URL.trim()
            def composeServices = sh(script: "docker-compose config --services", returnStdout: true).trim().split('\n')

            docker.withRegistry("https://${ecrRepoUrl}", 'ecr:us-west-2:aws-credentials') {
                composeServices.each { service ->

                    sh "docker tag ${service}:latest ${ecrRepoUrl}/benlmaoujoud:${service}-latest"
                    sh "docker push ${ecrRepoUrl}/benlmaoujoud:${service}-latest"
                }
            }
        }
    }
}
        stage('OWASP ZAP Scan') {
    steps {
        script {

             sh 'wget -q -O zap.sh https://github.com/zaproxy/zaproxy/releases/download/v2.10.0/ZAP_2.10.0_Crossplatform.zip'
             sh 'unzip zap.zip'
             sh 'chmod +x zap.sh'
    
                // Replace the following URL with the actual URL of your application
                def targetUrl = 'http://35.88.88.115:8080'
    
                // Run OWASP ZAP scanning
                sh "zap.sh -cmd -quickurl ${targetUrl} -quickout zap-report.html"
        }
    }
}

    }
}
