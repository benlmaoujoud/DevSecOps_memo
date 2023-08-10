pipeline {
    agent any

    stages {
        
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
}
