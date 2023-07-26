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
                    def services = ['product-service', 'order-service', 'inventory-service', 'discovery-server', 'api-gateway', 'notification-service']
                    def scanResults = [:]

                    for (service in services) {
                        def scanOutput = sh(script: "cd ${service} && mvn snyk:test -fn", returnStdout: true) || true
                        scanResults[service] = scanOutput.trim()
                    }

                    echo "Snyk scan results for each service:"
                    scanResults.each { service, output ->
                        echo "\nService: ${service}"
                        echo output
                    }
                }
            }
        }
    }
}
