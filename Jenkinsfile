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
                script {
                    def snykToken = '0c52b3b2-14ed-4d5c-bafb-9171793ead22'
                    def services = ['product-service', 'order-service', 'inventory-service', 'discovery-server', 'api-gateway', 'notification-service']

                    for (service in services) {
                        sh "cd ${service} && mvn snyk:test --token=${snykToken}"
                    }
                }
            }
        }
        
    }
}
