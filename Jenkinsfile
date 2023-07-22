pipeline {
  agent any
  tools {
        maven 'Maven_3_5_2'
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=ensias-internship -Dsonar.organization=ensias-internship -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=87868ea1485745b552e22260f9635d4bbab65d6c'
			}
        }
  }
}