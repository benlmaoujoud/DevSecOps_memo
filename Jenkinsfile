pipeline {
    agent any

    stages {
        stage('Run ZAP Security Scan') {
            steps {
                sh "zap.sh -cmd -quickurl http://35.88.88.115:8080 -quickout ${JENKINS_HOME}/workspace/zap-report.html"
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${JENKINS_HOME}/workspace/zap-report.html", allowEmptyArchive: true
        }
    }
}
