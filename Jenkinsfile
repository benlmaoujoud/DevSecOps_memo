pipeline {
    agent any

    tools {
        maven 'Maven3_5_2'
    }
    
    environment {
        AWS_REGION = 'us-west-2'
        ECR_REPO_URL = '079084503647.dkr.ecr.us-west-2.amazonaws.com'
    }

     
        stage('OWASP ZAP Scan') {
    steps {
        script {

        
                // Replace the following URL with the actual URL of your application
                def targetUrl = 'http://35.88.88.115:8080'
    
                // Run OWASP ZAP scanning
                sh "zap.sh -cmd -quickurl ${targetUrl} -quickout zap-report.html"
        }
    }
}

    }
}
